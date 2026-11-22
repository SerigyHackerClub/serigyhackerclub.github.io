---
ordem: 0
title: "Guia para iniciantes"
description: "Guia para iniciantes em cibersegurança: redes, sistemas operacionais, Linux, arquitetura e CTFs."
---

Este guia é do Serigy Hacker Club para quem quer entrar em cibersegurança do zero. Ele foi escrito para ser lido em ordem, mas cada página funciona sozinha. Não precisa saber programar para começar, embora um pouco de Python ajude bastante mais à frente.

## A trilha

{% include trilha.html %}

## Como estudar

- **Pratique mais do que leia.** Cada página termina com exercícios. Faça pelo menos um antes de seguir.
- **Use uma máquina virtual.** Assim você pode errar, quebrar e recomeçar sem arriscar o seu computador.
- **Anote tudo.** Um caderno em Markdown já basta. Explicar o que você aprendeu com as suas palavras fixa muito mais do que copiar.
- **Pergunte.** Ninguém aprende isso sozinho. Entre na comunidade e leve as suas dúvidas.

## Para processos seletivos e desafios

Muitos processos seletivos de segurança, como os de empresas que abrem vagas de trainee, e muitos desafios de CTF cobram as mesmas três coisas, e as páginas desta trilha cobrem as duas primeiras:

- **Redes:** IP, portas, TCP e UDP, DNS, HTTP e HTTPS, cliente e servidor, e análise de uma captura de tráfego no Wireshark.
- **Sistemas operacionais e Linux:** terminal, permissões, usuários e grupos, processos, variáveis de ambiente, serviços, busca por arquivos e textos, e noções de Docker.
- **Programação e lógica:** variáveis, condicionais, laços, funções, listas, strings, entrada e saída de dados, e resolução de problemas simples. Python é uma ótima escolha.

Algumas dicas que valem para todos esses desafios:

- **Entenda o que está sendo pedido antes de começar.** Em um desafio de analisar tráfego ou um servidor, as próprias perguntas servem de guia para os filtros e comandos que você vai usar.
- **Leia as mensagens de erro.** Muitas vezes elas dizem exatamente o que falta: o método, um cabeçalho, uma permissão.
- **Confira por dois caminhos.** Se você achou algo na interface gráfica, tente confirmar no terminal.
- **Registre o seu raciocínio.** Salve os comandos e filtros que usou. Quem avalia costuma pedir que você explique como chegou à resposta, e saber contar o caminho vale tanto quanto a resposta.
- **Pense como defensor também.** Para cada coisa que você achar (uma senha exposta, uma porta aberta, um binário estranho), pergunte qual é o risco e como corrigir.
- **Treine só em ambientes autorizados.** Use a sua VM, capturas de treino e laboratórios feitos para isso.
