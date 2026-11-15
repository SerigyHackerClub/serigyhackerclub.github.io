---
title: "Recon na prática: subdomínios, diretórios, DNS e virtual hosts"
date: 2026-11-12 10:00:00 -0300
description: "Quatro técnicas de reconhecimento em exercícios do PentesterLab, com os comandos e o raciocínio de cada uma."
tags: [recon, pentest]
author: "Lip Smith"
---

Reconhecimento (*recon*) é a fase em que você descobre o que existe antes de tentar qualquer coisa. Este post reúne quatro técnicas, usadas em exercícios de recon do PentesterLab contra o domínio de treino `hackycorp.com`.

Faça isso **somente** em alvos autorizados, como laboratórios e plataformas de treino. Varrer o site de terceiros sem permissão pode ser crime.

## 1. Descobrir subdomínios

Um domínio costuma ter mais sites do que o `www`. Ferramentas como o `subfinder` consultam fontes públicas (certificados, buscadores, bases de DNS) e listam os subdomínios conhecidos:

```bash
subfinder -d hackycorp.com
```

Saída do exercício:

```
66177e3f25e3ea0713807b1dc5f0b9df.hackycorp.com
assets.hackycorp.com
www.hackycorp.com
[INF] Found 3 subdomains for hackycorp.com in 1 second 477 milliseconds
```

Dois pontos de atenção. Um subdomínio com nome aleatório, como o primeiro, costuma existir por um motivo: vale visitar. E `assets` costuma servir arquivos estáticos, que às vezes guardam coisas que não deveriam estar ali.

## 2. Descobrir diretórios e arquivos com fuzzing

*Fuzzing* de diretórios é testar uma lista de nomes comuns contra o site e ver quais existem. O `ffuf` faz isso rápido. O `FUZZ` na URL é trocado por cada palavra da lista:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -u https://hackycorp.com/FUZZ
```

A lista `common.txt` vem no pacote SecLists. O resultado do exercício:

```
.well-known/security.txt [Status: 200, Size: 156]
admin                    [Status: 301, Size: 178]
images                   [Status: 301, Size: 178]
index.html               [Status: 200, Size: 16011]
robots.txt               [Status: 200, Size: 121]
startpage                [Status: 301, Size: 178]
```

Como ler:

- **200** é uma página que existe. **301** é um redirecionamento, normalmente de uma pasta (o servidor quer a barra no final). **401** e **403** indicam que existe, mas você não tem acesso, e isso também é informação.
- `startpage` não é um nome que se acharia olhando o site, e foi justamente ele que levou ao próximo passo do exercício.
- Olhe o `robots.txt` e o `.well-known/security.txt`: o primeiro pode listar caminhos que o dono quer esconder, e o segundo diz como reportar falhas.

O `ffuf` manda centenas de requisições por segundo. Em um alvo real e autorizado, combine o ritmo com o responsável, para não derrubar nada.

## 3. DNS e acesso direto pelo IP

O `dig` mostra o que o DNS responde para um nome:

```bash
dig hackycorp.com
```

Na seção `ANSWER`, o registro `A` traz o IP do servidor. O exercício seguinte é simples e instrutivo: acessar o servidor **pelo IP**, em vez do nome:

```bash
curl IP_DO_SERVIDOR
```

A resposta foi diferente da do site normal. Isso acontece porque um servidor pode responder de um jeito quando é chamado pelo IP e de outro quando é chamado pelo nome. Por isso, sempre teste o IP descoberto.

## 4. Virtual hosts: o cabeçalho Host

Um mesmo IP pode hospedar vários sites. O servidor decide qual entregar olhando o cabeçalho `Host` da requisição HTTP. Se você conhece (ou adivinha) um nome que o servidor atende, pode pedir esse site diretamente, mesmo que o DNS desse nome não aponte para lá:

```bash
curl -H "Host: olhamensagem.com" https://IP_DO_SERVIDOR --insecure -v
```

O que cada parte faz:

- `-H "Host: ..."` troca o cabeçalho `Host` da requisição.
- `--insecure` ignora a validação do certificado. O certificado do servidor é do `hackycorp.com`, então o nome não bate. Use isso apenas em laboratório.
- `-v` mostra a conversa inteira, inclusive a negociação TLS e os cabeçalhos.

Uma forma mais limpa é o `--resolve`, que faz o nome apontar para o IP só nessa requisição e envia o nome certo também na negociação TLS (SNI):

```bash
curl --resolve olhamensagem.com:443:IP_DO_SERVIDOR https://olhamensagem.com -k
```

Para descobrir nomes de virtual hosts que você não conhece, o próprio `ffuf` ajuda: troque o `Host` pela palavra da lista (`-H "Host: FUZZ.dominio.com"`) e observe quais respostas têm tamanho diferente.

## Leia os cabeçalhos da resposta

Com o `-v`, a resposta do exercício mostrou, além da página, um cabeçalho que não é padrão:

```
< Server: nginx
< Last-Modified: Wed, 01 Apr 2020 03:25:09 GMT
< pentesterlab_recon_09: (valor omitido)
```

Cabeçalhos fora do comum entregam informação: tecnologia do servidor, versões, nomes internos, identificadores. Em um teste de verdade, você anota todos.

## Resumo

| Técnica | Ferramenta | Pergunta que responde |
|---|---|---|
| Subdomínios | `subfinder` | Que outros sites esse domínio tem? |
| Diretórios | `ffuf` | Que páginas e pastas existem? |
| DNS | `dig` | Para qual IP esse nome aponta? |
| Acesso por IP | `curl` | O servidor responde diferente sem o nome? |
| Virtual hosts | `curl -H "Host: ..."` | Que sites vivem nesse mesmo IP? |
| Cabeçalhos | `curl -v` | O que o servidor entrega sem querer? |

## Para praticar

- [PentesterLab](https://pentesterlab.com/): os exercícios de recon usados aqui.
- [SecLists](https://github.com/danielmiessler/SecLists): as listas de palavras para fuzzing.
- [ffuf](https://github.com/ffuf/ffuf) e [subfinder](https://github.com/projectdiscovery/subfinder): documentação das ferramentas.
