---
ordem: 5
title: "CTFs: da primeira flag à primeira competição"
description: "CTFs para iniciantes: formatos, categorias, ferramentas e como jogar a primeira competição."
---

CTF significa *Capture The Flag*. São desafios de segurança em que o objetivo é encontrar uma **flag**, um texto secreto como `flag{exemplo}`, e enviá-la para ganhar pontos. É a melhor forma de praticar, porque você aprende fazendo e de forma legal.

## Formatos

- **Jeopardy:** uma lista de desafios por categoria, cada um com pontuação. É o formato mais comum e o melhor para começar.
- **Attack-Defense:** cada equipe tem um serviço vulnerável para proteger e, ao mesmo tempo, ataca os das outras.
- **King of the Hill:** as equipes disputam o controle de uma máquina.
- **Wargames:** níveis em sequência que você vai destravando, como o Bandit.

## Categorias

| Categoria | O que você faz | Ferramentas comuns |
|---|---|---|
| Web | Acha falhas em sites e APIs | Navegador (DevTools), Burp Suite, curl |
| Criptografia | Quebra ou entende cifras | Python, CyberChef |
| Forense | Analisa arquivos, discos, tráfego e imagens | Wireshark, `binwalk`, `exiftool`, `file` |
| Engenharia reversa | Entende como um programa funciona | Ghidra, `strings`, `objdump`, `gdb` |
| Pwn | Explora falhas em binários | `gdb` e pwntools |
| OSINT | Pesquisa em fontes abertas | Buscadores, mapas, redes sociais |
| Misc / Stego | Desafios variados, dados escondidos | Um pouco de tudo |

## Por onde começar

Uma ordem que costuma funcionar:

1. **Linux** pelo terminal (o Bandit, do OverTheWire, é ideal).
2. **Web básico**: HTTP, formulários, cookies, injeções.
3. **Criptografia clássica e forense**: encodings, cifras simples, metadados.
4. **Engenharia reversa**: ler binários pequenos.
5. **Pwn**, que pede a base de arquitetura da outra página deste guia.

Você não precisa dominar tudo. A maioria das equipes se divide por categoria.

## Ferramentas que valem instalar

- **Burp Suite Community** e as DevTools do navegador, para web.
- **CyberChef**, o "canivete suíço" para decodificar, converter e testar encodings.
- **Wireshark**, para ler capturas de rede.
- **Ghidra**, para ler binários.
- **Python** com a biblioteca **pwntools**, para automatizar.
- Utilitários de linha de comando: `file`, `strings`, `binwalk`, `exiftool`, `xxd`.

## Onde treinar

- [picoCTF](https://picoctf.org/): feito para iniciantes, com desafios de todas as categorias.
- [OverTheWire](https://overthewire.org/wargames/): wargames, com o Bandit para começar.
- [PortSwigger Web Security Academy](https://portswigger.net/web-security): o melhor curso gratuito de segurança web.
- [CryptoHack](https://cryptohack.org/): criptografia na prática.
- [pwn.college](https://pwn.college/): exploração de binários, do zero.
- [TryHackMe](https://tryhackme.com/) e [Hack The Box](https://www.hackthebox.com/): salas e máquinas guiadas.
- [CTFtime](https://ctftime.org/): calendário das competições, com filtros para as mais acessíveis.

## Como jogar uma competição

1. **Leia as regras** antes de começar: formato da flag, limite de equipe, o que é proibido.
2. **Olhe todos os desafios** e comece pelos que mais equipes resolveram. Eles costumam ser os mais fáceis.
3. **Leia o enunciado duas vezes.** Muitos desafios dão a dica no texto.
4. **Anote tudo**: o que você tentou, o que deu errado, o que descobriu.
5. **Não fique preso.** Se passou de uns 30 a 40 minutos sem avançar, troque de desafio ou peça ajuda.
6. **Respeite o fair play**: não ataque a infraestrutura do evento, não compartilhe flags durante a competição e não faça força bruta onde as regras proíbem.

## Jogando em equipe

- Dividam as categorias por afinidade.
- Usem um canal de chat e um documento compartilhado com as anotações.
- Avisem quando começarem um desafio, para duas pessoas não fazerem o mesmo.
- Ensinem uns aos outros. O iniciante que pergunta ajuda a equipe toda a revisar o raciocínio.

## Writeups

Um **writeup** é o relato de como você resolveu um desafio. Escrever um é a melhor forma de fixar o que aprendeu e ainda ajuda a comunidade. Uma estrutura simples:

1. O desafio e o que ele pedia.
2. Como você analisou e o que tentou.
3. A solução, passo a passo.
4. O que você aprendeu.

Publique depois do fim do evento e confira se as regras permitem.

## Seu primeiro mês de CTF

1. Chegue ao nível 15 do **Bandit**.
2. Resolva os desafios de *General Skills* do **picoCTF**.
3. Escolha no **CTFtime** uma competição com bom acesso para iniciantes e jogue com uma equipe.
4. Escreva o seu primeiro writeup.

O Serigy Hacker Club incentiva CTFs em Sergipe. Fique de olho nos nossos canais para saber das próximas competições e montar equipe.
