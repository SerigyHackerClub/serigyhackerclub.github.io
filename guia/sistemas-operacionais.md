---
ordem: 2
title: "Sistemas operacionais"
description: "Sistemas operacionais para iniciantes: kernel, processos, memória e permissões."
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

Um sistema de arquivos (ext4 no Linux, NTFS no Windows) organiza os dados em diretórios e arquivos. Cada arquivo tem um dono e permissões de leitura, escrita e execução. Existe um usuário todo-poderoso: `root` no Linux e `Administrator` (ou `SYSTEM`) no Windows. Boa parte da segurança é manter os programas rodando com o **mínimo de privilégio** necessário.

## Linux e Windows, lado a lado

| | Linux | Windows |
|---|---|---|
| Caminhos | `/home/ana/foto.png` | `C:\Users\ana\foto.png` |
| Configuração | Arquivos em `/etc` | Registro e arquivos |
| Serviços | `systemd` (`systemctl`) | Serviços (`services.msc`) |
| Superusuário | `root` | `Administrator` / `SYSTEM` |
| Logs | `/var/log`, `journalctl` | Visualizador de Eventos |

## Mão na massa

```bash
ps aux                       # todos os processos
top                          # uso de CPU e memória em tempo real
cat /proc/self/maps | head   # regiões de memória do próprio processo
ls -l /proc/$$/fd            # arquivos abertos pelo seu shell
strace ls                    # as syscalls que o comando ls faz
```

No Windows, use o Gerenciador de Tarefas ou o Process Explorer.

Exercícios:

1. Abra um programa e ache o processo dele com `ps`. Anote o PID e mate-o com `kill`.
2. Rode `strace ls` e identifique as chamadas `openat` e `write`.
3. Rode `cat /proc/self/maps` duas vezes e compare os endereços. Eles mudaram? Por quê?

## Para aprofundar

- [Operating Systems: Three Easy Pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/): livro gratuito, muito didático.
- [Wizard Zines](https://wizardzines.com/): zines sobre Linux e processos.
