---
layout: post
title: "Writeup: pós-exploração em Active Directory, do usuário comum ao Domain Admin"
description: "Como um usuário sem privilégios administrativos chegou a Domain Admin num laboratório de AD: AS-REP Roasting, abuso de ACLs, DCSync e Golden Ticket."
tags: [active-directory, pentest, windows, writeup]
author: "Lip Smith"
---

Este é o writeup da atividade final do módulo de Pós-Exploração do treinamento de Offensive Security da Pacific Sec. O desafio seguia o modelo **assume breach**: eu recebia acesso a uma máquina Windows já dentro do domínio `ad.lab.test`, com um usuário local sem nenhum privilégio administrativo. A pergunta era simples de enunciar e difícil de responder: esse usuário conseguiria, a partir do zero, chegar a um arquivo restrito à administração e extrair um valor específico dele?

**Ambiente:** laboratório isolado de treinamento, domínio fictício `ad.lab.test`. Nada aqui foi feito fora de um ambiente autorizado.

## O objetivo

Acessar `\\DC01\pessoal\cadastro_colaboradores_2026.csv` e obter o valor do campo `codigo_lote`. Ferramentas: Firefox, BloodHound Community Edition, SharpHound, PowerView, Impacket, GetNPUsers, secretsdump, smbclient, Python 3.12 e Mimikatz.

## Reconhecimento

O primeiro passo foi mapear onde eu estava:

```powershell
whoami /all
ipconfig /all
ping DC01.AD.LAB.TEST
net accounts
```

Daí saíram duas informações que mudaram o resto da investigação: o IP do controlador de domínio (`10.80.6.4`) e o `Lockout threshold: Never`. Sem bloqueio de conta, eu sabia que mais à frente poderia tentar senhas sem me preocupar em travar contas. Também encontrei, na própria máquina, um arquivo `usuarios.txt` com nomes de usuários do domínio, que acabou sendo a lista usada no próximo passo.

## AS-REP Roasting: a primeira credencial sem senha nenhuma

Com a lista de usuários em mãos, testei um **AS-REP Roasting**: um ataque contra o Kerberos que extrai o hash de contas configuradas com a flag `UF_DONT_REQUIRE_PREAUTH` (pré-autenticação desabilitada), sem precisar de nenhuma credencial válida antes.

```
GetNPUsers.py ad.lab.test/ -usersfile usuarios.txt -no-pass -dc-ip 10.80.6.4 \
  -format hashcat -outputfile asrep_hashes.txt
```

A conta `igor.rezende` estava vulnerável. Levei o hash para o John the Ripper, com o dicionário `rockyou.txt`, e a senha caiu: `Batman123`. Com isso, eu já tinha uma identidade de domínio.

## Enumerando o domínio com BloodHound

Com uma conta válida, rodei o **SharpHound** para coletar a estrutura do AD e importei no **BloodHound CE**:

```
SharpHound.exe -c All -d AD.LAB.TEST
```

O BloodHound mapeou uma cadeia de permissões mal configuradas: `igor.rezende` tinha `GenericWrite` sobre um grupo, que tinha `WriteOwner` sobre outro, que tinha `ForceChangePassword` sobre a conta `beatriz.moreira`. Ou seja, dava para seguir essa corrente até forçar a troca de senha dela, sem nunca saber a senha original.

Com o PowerView, segui a corrente: entrei no primeiro grupo, me tornei dono do segundo, e troquei a senha da `beatriz.moreira`. O detalhe que me chamou atenção foi uma anotação no próprio BloodHound dizendo que essa conta "opera arquivos do setor de suporte no file server". Isso bastou para eu ir direto atrás do compartilhamento de suporte.

## Credenciais em texto claro

No compartilhamento, um arquivo `readme_credenciais.txt` guardava, sem nenhuma proteção, a senha de uma conta de serviço:

```
Conta de serviço de suporte: elena.teixeira
Senha atual: Aa1!yeqyV1CFV5OqrCQj
(rotacionar trimestralmente – pendente)
```

Voltando ao BloodHound com essa nova conta, apareceu o caminho que fechava a exploração: `elena.teixeira` tinha `GenericAll` sobre um grupo que, por sua vez, tinha a permissão de **DCSync** no domínio inteiro. `GenericAll` dá controle total sobre o objeto, então bastou adicionar a `elena.teixeira` a esse grupo.

## DCSync e o acesso final

Com a permissão de DCSync ativa, usei o `secretsdump.py` para extrair o hash NTLM de um Domain Admin, `joao.freitas`, sem precisar tocar diretamente no controlador de domínio:

```
secretsdump.py ad.lab.test/elena.teixeira:'Aa1!yeqyV1CFV5OqrCQj'@10.80.6.4 -just-dc-user joao.freitas
```

Com o hash em mãos, fiz um **Pass-the-Hash** usando o `smbclient.py`, autenticando como `joao.freitas` sem nunca saber a senha dele em texto claro, e baixei o arquivo-alvo. O valor de `codigo_lote` era `PII-2BF95445056F`.

## Indo além: Golden Ticket

Já com DCSync ativo, fui um pouco além do pedido e extraí também o hash da conta `krbtgt`, que assina todos os tickets Kerberos do domínio. Com esse hash e o SID do domínio, forjei um **Golden Ticket** via Mimikatz, assumindo a identidade do `Administrator` com os grupos de Domain Admins, Enterprise Admins e Schema Admins:

```
kerberos::golden /user:Administrator /domain:ad.lab.test /sid:S-1-5-21-... \
  /krbtgt:5ad760c20f63cc74da8cce60dda0b0c3 /ptt
```

O `klist` confirmou dois detalhes que mostram bem o estrago que isso causa: o ticket tinha validade até **2036** (10 anos, o padrão do Mimikatz), e o campo `Kdc Called` estava vazio, ou seja, o ticket foi aceito localmente sem nenhuma requisição ao controlador de domínio. Isso é o que torna um Golden Ticket tão perigoso: ele não deixa rastro de autenticação nos logs do DC, e uma única troca de senha do `krbtgt` não invalida tickets já emitidos. É preciso trocar a senha **duas vezes seguidas**.

## O que ficou de aprendizado

A cadeia inteira, do primeiro acesso ao Domain Admin, não usou uma única vulnerabilidade de software. Foi tudo configuração:

- uma conta sem pré-autenticação Kerberos;
- permissões de delegação (`GenericWrite`, `WriteOwner`, `ForceChangePassword`) encadeadas entre grupos de departamentos diferentes;
- uma senha em texto claro num compartilhamento de rede;
- uma permissão de DCSync concedida a um grupo que não deveria tê-la.

Cada passo isolado parece pequeno. Juntos, levam de um usuário comum a dono do domínio. Isso é basicamente o motivo de existir o BloodHound: sozinho, um administrador dificilmente vê essas cadeias; em grafo, elas pulam aos olhos.

As principais mitigações, para quem quiser aplicar num AD de verdade:

1. Habilitar a pré-autenticação Kerberos em todas as contas.
2. Revisar ACLs entre grupos periodicamente, com BloodHound ou `Get-DomainObjectAcl`.
3. Nunca guardar credenciais em texto claro em compartilhamentos; usar um cofre de senhas.
4. Restringir DCSync a Domain Admins e Domain Controllers, e nada além disso.
5. Habilitar lockout de conta e uma política de senha forte.
6. Usar LAPS para rotacionar senhas de administrador local.
7. Lembrar que, após um comprometimento com Golden Ticket, a senha do `krbtgt` precisa ser trocada duas vezes.

Esse foi o meu primeiro contato a fundo com abuso de ACLs em Active Directory de ponta a ponta, e o BloodHound virou, sem dúvida, minha ferramenta favorita da semana.
