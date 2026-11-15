---
ordem: 3
title: "Linux para iniciantes"
description: "Linux para iniciantes: terminal, comandos essenciais, permissões e scripts."
---

Linux está nos servidores, nas ferramentas de segurança e na maioria dos desafios de CTF. Ficar à vontade no terminal é a habilidade que mais rende no começo.

## Como começar

- **Máquina virtual:** instale o VirtualBox ou o VMware e crie uma VM com Ubuntu ou Debian. É o jeito mais seguro de treinar.
- **WSL2:** se você usa Windows, dá para ter um Linux dentro dele.
- **Distribuição:** comece com Ubuntu ou Debian. O Kali Linux, cheio de ferramentas de segurança, fica para depois, quando você já souber o básico.

## Onde as coisas ficam

| Diretório | Conteúdo |
|---|---|
| `/` | A raiz de tudo |
| `/home` | Os arquivos dos usuários |
| `/etc` | Arquivos de configuração |
| `/var/log` | Logs do sistema |
| `/tmp` | Arquivos temporários |
| `/usr/bin` | Programas |
| `/proc` | Informações dos processos, geradas pelo kernel |
| `/root` | A pasta do superusuário |

## Comandos essenciais

| Comando | O que faz |
|---|---|
| `pwd` | Mostra onde você está |
| `ls -la` | Lista arquivos, inclusive os ocultos |
| `cd pasta` | Entra em uma pasta |
| `cat arquivo` | Mostra o conteúdo |
| `less arquivo` | Lê com rolagem (saia com `q`) |
| `head` / `tail -f` | Início / fim do arquivo, e `-f` acompanha ao vivo |
| `cp`, `mv`, `rm` | Copiar, mover e apagar |
| `mkdir`, `touch` | Criar pasta e arquivo |
| `find` | Procurar arquivos |
| `grep` | Procurar texto |
| `man comando` | Manual do comando |

Cuidado: o `rm` apaga sem lixeira, e `rm -rf` em um caminho errado pode destruir muita coisa. Releia o comando antes de apertar Enter.

## Permissões

Ao rodar `ls -l`, você vê algo como `-rwxr-xr--`. Leia em três blocos:

- `rwx` para o **dono**: ler, escrever, executar.
- `r-x` para o **grupo**: ler e executar.
- `r--` para **os outros**: só ler.

Cada permissão vale um número (r = 4, w = 2, x = 1). Somando, `rwx` é 7, `r-x` é 5 e `r--` é 4, então esse arquivo é `754`. Para mudar:

```bash
chmod 755 script.sh     # dono rwx, grupo e outros r-x
chmod +x script.sh      # dá permissão de execução
chown ana:ana arquivo   # muda dono e grupo
sudo comando            # executa com privilégio de administrador
```

## Pipes e redirecionamento

O poder do terminal está em combinar comandos pequenos:

```bash
ls -la > lista.txt              # grava a saída em um arquivo
echo "oi" >> lista.txt          # acrescenta ao fim
cat /etc/passwd | cut -d: -f1   # lista só os nomes de usuário
grep -i erro /var/log/syslog | wc -l   # conta as linhas com "erro"
```

O `|` (pipe) liga a saída de um comando à entrada do próximo.

## Processos, serviços e pacotes

```bash
ps aux | grep ssh          # procura um processo
kill 1234                  # encerra o processo de PID 1234
systemctl status ssh       # estado de um serviço
journalctl -u ssh -n 20    # últimas linhas do log do serviço
sudo apt update && sudo apt install nmap   # instala um programa (Debian/Ubuntu)
```

## Um script em bash

```bash
#!/bin/bash
for i in 1 2 3; do
  echo "Tentativa $i"
done
```

Salve como `teste.sh`, dê `chmod +x teste.sh` e rode com `./teste.sh`.

## Mão na massa

1. Crie uma pasta, entre nela e crie três arquivos. Liste com `ls -la`.
2. Tire a permissão de leitura de um dos arquivos e tente ler. Leia a mensagem de erro.
3. Conte quantos usuários existem em `/etc/passwd` usando um pipe.
4. Jogue os primeiros níveis do **Bandit**, do OverTheWire. É o melhor treino para o terminal.

## Para aprofundar

- [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/): jogo para aprender Linux.
- [Linux Journey](https://linuxjourney.com/): curso gratuito, passo a passo.
- [The Linux Command Line](https://linuxcommand.org/tlcl.php): livro gratuito.
