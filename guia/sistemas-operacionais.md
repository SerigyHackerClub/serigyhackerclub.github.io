---
ordem: 2
title: "Sistemas operacionais"
description: "Sistemas operacionais para iniciantes: kernel, processos, memória, usuários, permissões, serviços e containers."
---

O sistema operacional é o programa que fica entre o hardware e todos os outros programas. Entender o que ele faz explica por que existem permissões, por que um programa "trava" sem derrubar o computador e onde muitos ataques acontecem.

## O que um sistema operacional faz

- **Gerencia processos:** decide qual programa usa a CPU e por quanto tempo.
- **Gerencia a memória:** entrega a cada programa o seu espaço e impede que um leia o do outro.
- **Gerencia arquivos:** organiza os dados no disco.
- **Controla dispositivos:** teclado, rede, disco, tela, por meio de drivers.
- **Controla o acesso:** usuários, grupos e permissões.

## Kernel, modo usuário e modo kernel

O **kernel** é o núcleo do sistema. Ele roda com privilégio total, no chamado *modo kernel*. Os programas comuns rodam em *modo usuário*, com limites. Quando um programa precisa de algo que só o kernel pode fazer, como abrir um arquivo ou enviar dados pela rede, ele faz uma **chamada de sistema** (*syscall*), como `open`, `read`, `write` e `execve`.

Essa divisão é a base da segurança do sistema: um programa comum não deveria conseguir mexer direto no hardware ou na memória de outro.

## Processos e threads

Um **processo** é um programa em execução, com um identificador (PID), o seu espaço de memória e seus recursos. Uma **thread** é uma linha de execução dentro de um processo, e várias threads do mesmo processo compartilham a memória.

O **escalonador** (*scheduler*) alterna rapidamente entre os processos, e por isso parece que tudo roda ao mesmo tempo. No Linux, um processo cria outro com `fork` e troca o programa com `exec`. No Windows, a função equivalente é a `CreateProcess`.

Todo processo roda **em nome de um usuário**, e herda as permissões dele. Um servidor web rodando como `root` é muito mais perigoso do que um rodando com um usuário comum, porque quem invadir o processo ganha os mesmos poderes dele.

## Variáveis de ambiente

Cada processo recebe do pai um conjunto de pares `NOME=valor`, as **variáveis de ambiente**. Elas guardam configurações, como o caminho dos programas (`PATH`), o diretório do usuário (`HOME`) e quem é o usuário (`USER`). Os filhos herdam as variáveis do pai.

Por serem fáceis de passar para um programa, muita gente guarda **senhas e chaves de API** nelas. Isso é melhor do que escrever no código, mas não é um cofre: o próprio usuário (e o administrador) consegue ler o ambiente de um processo em execução. No Linux, ele fica visível em `/proc/<PID>/environ`.

## Memória

Cada processo enxerga uma **memória virtual** só dele. O sistema traduz esses endereços para a memória física, usando paginação. Isso isola os processos entre si.

O espaço de um processo costuma ter estas regiões:

| Região | Guarda |
|---|---|
| Texto (código) | As instruções do programa |
| Dados | Variáveis globais |
| Heap | Memória alocada dinamicamente |
| Stack (pilha) | Variáveis locais e endereços de retorno |

Duas proteções que você vai encontrar em CTFs: o **NX** marca regiões como não executáveis, e o **ASLR** muda os endereços a cada execução, para dificultar a vida de quem ataca.

## Arquivos, usuários e permissões

Um sistema de arquivos (ext4 no Linux, NTFS no Windows) organiza os dados em diretórios e arquivos. Cada arquivo tem um **dono**, um **grupo** e permissões de leitura, escrita e execução.

- **Usuário:** tem um nome e um número, o **UID**. No Linux, o `root` é sempre UID 0. Contas de pessoas costumam começar em 1000, e as de sistema e de serviço (como `www-data`) ficam abaixo disso.
- **Grupo:** reúne usuários para dar a mesma permissão a todos. Cada um tem um número, o **GID**. Um usuário pode estar em vários grupos.
- **Conta de serviço:** conta criada só para um programa rodar (o serviço de backup, o servidor web). Ela não é uma pessoa e deveria ter só as permissões de que o serviço precisa.
- **Superusuário:** `root` no Linux e `Administrator` (ou `SYSTEM`) no Windows.

Boa parte da segurança é aplicar o **princípio do mínimo privilégio**: cada usuário e cada programa recebe só o acesso de que precisa, e nada além disso.

### Permissões especiais: o SUID

Normalmente, um programa roda com os poderes de **quem o executou**. Um binário com o bit **SUID** ligado roda com os poderes do **dono do arquivo**. É assim que o `passwd` deixa um usuário comum trocar a própria senha, que fica em um arquivo só do root.

O problema é que, se um programa SUID de dono `root` tiver uma falha (como injeção de comando ou *buffer overflow*), o atacante ganha um shell de root. Por isso, binários SUID fora do padrão do sistema merecem atenção em qualquer auditoria.

## Serviços e tarefas agendadas

Um **serviço** (ou *daemon*) é um programa que roda em segundo plano, sem ninguém na frente, geralmente esperando pedidos em uma porta: servidor web, SSH, banco de dados. O sistema inicia, para e reinicia os serviços por você (no Linux, o `systemd`).

Um serviço **escutando em todas as interfaces** (`0.0.0.0`) aceita conexões de qualquer lugar da rede. Um serviço escutando só em `127.0.0.1` só é acessível da própria máquina. Em segurança, a pergunta é sempre: esse serviço precisa estar exposto?

Já as **tarefas agendadas** (no Linux, o **cron**; no Windows, o Agendador de Tarefas) executam um programa em um horário. São úteis para backup e limpeza, e também são um lugar clássico para atacantes se esconderem e para o administrador esquecer um script antigo com permissões frouxas.

## Containers

Um **container** é um jeito de rodar um programa isolado, com os seus próprios arquivos, processos e rede, **sem** precisar de um sistema operacional completo por trás. Diferentemente de uma máquina virtual, todos os containers **compartilham o kernel** do sistema onde rodam. Por isso são leves e iniciam em segundos.

O isolamento vem de recursos do próprio kernel Linux: *namespaces* (cada container enxerga só os seus processos, a sua rede e os seus arquivos) e *cgroups* (limitam a CPU e a memória que ele pode usar).

Vocabulário essencial do **Docker**, a ferramenta mais usada:

- **Imagem:** o "molde" de leitura, com o programa e tudo que ele precisa (arquivos, bibliotecas, configuração).
- **Container:** uma imagem em execução.
- **Dockerfile:** a receita de texto para construir uma imagem.
- **Volume:** pasta que guarda dados fora do container, para sobreviverem quando ele for apagado.
- **Porta publicada:** liga uma porta da sua máquina a uma porta dentro do container.

Muitos laboratórios e desafios de CTF rodam em containers, então saber entrar em um, ver os seus processos e ler os seus logs é útil desde cedo.

## Linux e Windows, lado a lado

| | Linux | Windows |
|---|---|---|
| Caminhos | `/home/ana/foto.png` | `C:\Users\ana\foto.png` |
| Configuração | Arquivos em `/etc` | Registro e arquivos |
| Serviços | `systemd` (`systemctl`) | Serviços (`services.msc`) |
| Tarefas agendadas | `cron` | Agendador de Tarefas |
| Variáveis de ambiente | `env`, `export` | `set`, Propriedades do Sistema |
| Superusuário | `root` | `Administrator` / `SYSTEM` |
| Logs | `/var/log`, `journalctl` | Visualizador de Eventos |

## Mão na massa

```bash
ps aux                       # todos os processos
top                          # uso de CPU e memória em tempo real
cat /proc/self/maps | head   # regiões de memória do próprio processo
ls -l /proc/$$/fd            # arquivos abertos pelo seu shell
strace ls                    # as syscalls que o comando ls faz
id                           # seu UID, GID e grupos
env                          # variáveis de ambiente do seu shell
cat /proc/self/environ | tr '\0' '\n'   # o ambiente do processo, via /proc
```

No Windows, use o Gerenciador de Tarefas ou o Process Explorer.

Exercícios:

1. Abra um programa e ache o processo dele com `ps`. Anote o PID e mate-o com `kill`.
2. Rode `strace ls` e identifique as chamadas `openat` e `write`.
3. Rode `cat /proc/self/maps` duas vezes e compare os endereços. Eles mudaram? Por quê?
4. Rode `id` e descubra o seu UID e os seus grupos. Depois olhe `/etc/passwd` e `/etc/group` e identifique a sua linha em cada um.
5. Defina uma variável com `export SEGREDO=abc123`, abra um processo filho (`bash`) e confira com `echo $SEGREDO` que ele herdou. Depois rode `sleep 300 &` e leia o ambiente dele em `/proc/<PID>/environ`.
6. Rode `docker run --rm -it ubuntu bash` e, dentro dele, rode `ps aux`. Quantos processos aparecem? E no seu computador? Por que a diferença?

## Para aprofundar

- [Operating Systems: Three Easy Pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/): livro gratuito, muito didático.
- [Wizard Zines](https://wizardzines.com/): zines sobre Linux e processos.
- [Docker: Getting Started](https://docs.docker.com/get-started/): o tutorial oficial.
- [Red Hat: O que é Docker](https://www.redhat.com/pt-br/topics/containers/what-is-docker): visão geral em português.
