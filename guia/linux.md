---
ordem: 3
title: "Linux para iniciantes"
description: "Linux para iniciantes: terminal, comandos essenciais, usuários, permissões, busca, serviços, scripts e Docker."
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
| `/usr/local/bin` | Programas instalados pelo administrador (fora do gerenciador de pacotes) |
| `/proc` | Informações dos processos, geradas pelo kernel |
| `/root` | A pasta do superusuário |

## Comandos essenciais

| Comando | O que faz |
|---|---|
| `pwd` | Mostra onde você está |
| `ls -la` | Lista arquivos, inclusive os ocultos |
| `cd pasta` | Entra em uma pasta (`cd ..` sobe, `cd` volta para o seu home) |
| `cat arquivo` | Mostra o conteúdo |
| `less arquivo` | Lê com rolagem (saia com `q`) |
| `head` / `tail -f` | Início / fim do arquivo, e `-f` acompanha ao vivo |
| `cp`, `mv`, `rm` | Copiar, mover e apagar |
| `mkdir`, `touch` | Criar pasta e arquivo |
| `find` | Procurar arquivos |
| `grep` | Procurar texto |
| `stat arquivo` | Dono, grupo, permissões e datas do arquivo |
| `man comando` | Manual do comando |

Cuidado: o `rm` apaga sem lixeira, e `rm -rf` em um caminho errado pode destruir muita coisa. Releia o comando antes de apertar Enter.

Dica de leitura: `ls -la` mostra os **arquivos ocultos**, que começam com ponto (`.bash_history`, `.env`, `.ssh`). É neles que costumam ficar configurações e, às vezes, coisas que não deveriam estar ali.

## Usuários e grupos

```bash
whoami                  # seu nome de usuário
id                      # seu UID, GID e todos os seus grupos
groups                  # só os grupos
id ana                  # o mesmo, para outro usuário
cat /etc/passwd         # lista de contas
cat /etc/group          # lista de grupos e seus membros
sudo -l                 # o que VOCÊ pode rodar com sudo
cat /etc/os-release     # qual distribuição e versão está rodando
```

Cada linha do `/etc/passwd` tem campos separados por dois-pontos:

```text
ana:x:1001:1001:Ana Souza:/home/ana:/bin/bash
 |  |  |    |      |         |         └ shell
 |  |  |    |      |         └ diretório home
 |  |  |    |      └ nome completo (comentário)
 |  |  |    └ GID (grupo principal)
 |  |  └ UID
 |  └ a senha fica no /etc/shadow
 └ nome de usuário
```

- **Contas de pessoas** costumam ter UID a partir de 1000, um home em `/home` e um shell de verdade (`/bin/bash`). **Contas de sistema e de serviço** têm UID baixo e geralmente `/usr/sbin/nologin` no lugar do shell. Para listar só as contas "reais": `awk -F: '$3 >= 1000 && $3 < 65534' /etc/passwd`.
- As **senhas** ficam em `/etc/shadow`, legível só pelo root. Cada senha é guardada como um *hash*, e o prefixo diz o algoritmo: `$6$` é SHA-512, `$5$` é SHA-256 e `$y$` é yescrypt.
- Um usuário pode usar `sudo` se estiver no grupo `sudo` (Debian/Ubuntu) ou `wheel` (outras distros), ou se aparecer no arquivo `/etc/sudoers`. Para ver quem está no grupo: `getent group sudo`.

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

Para ver o número direto, em vez de ler o `rwx`, use o `stat`:

```bash
stat -c '%a %U:%G %n' arquivo   # 754 ana:ana arquivo
```

Um arquivo com permissão `644` pode ser lido por **qualquer usuário** do sistema. Se ele guarda uma senha ou um log com dados sensíveis, isso é um problema.

### SUID e SGID

Quando o bit **SUID** está ligado, o arquivo executável roda com os poderes do **dono** dele, e não de quem o chamou. No `ls -l` ele aparece como um `s` no lugar do `x` do dono: `-rwsr-xr-x`. O **SGID** faz o mesmo com o grupo.

```bash
find / -perm -4000 -type f 2>/dev/null    # lista todos os binários com SUID
```

Vários binários do sistema têm SUID de propósito (`passwd`, `sudo`, `su`). O que merece investigação é um binário **customizado**, em lugares como `/usr/local/bin` ou `/opt`, com dono `root` e SUID ligado.

## Pipes e redirecionamento

O poder do terminal está em combinar comandos pequenos:

```bash
ls -la > lista.txt              # grava a saída em um arquivo
echo "oi" >> lista.txt          # acrescenta ao fim
cat /etc/passwd | cut -d: -f1   # lista só os nomes de usuário
grep -i erro /var/log/syslog | wc -l   # conta as linhas com "erro"
```

O `|` (pipe) liga a saída de um comando à entrada do próximo. O `2>/dev/null` joga as mensagens de erro fora (útil no `find`, que reclama de cada pasta sem permissão).

## Buscando arquivos e textos

Dois comandos que você vai usar o tempo todo:

```bash
find / -name "app.conf" 2>/dev/null        # por nome, no sistema todo
find /home -type f -user ana               # arquivos de um usuário
find / -type f -mtime -2 2>/dev/null       # modificados nos últimos 2 dias
find / -type f -perm -4000 2>/dev/null     # arquivos com SUID

grep -r "senha" /etc 2>/dev/null           # procura texto dentro de arquivos, recursivamente
grep -rin "password" /home/ana             # -i ignora maiúsculas, -n mostra o número da linha
grep -rl "token" /opt                      # -l mostra só o nome dos arquivos
```

Combine os dois: `find /opt -name "*.conf" | xargs grep -i "user"`.

## Variáveis de ambiente

```bash
env                      # lista todas as variáveis do seu shell
echo $HOME               # valor de uma variável
export CHAVE=valor       # cria uma variável (os processos filhos herdam)
```

Para ver o ambiente de **outro processo** em execução, leia o arquivo dele em `/proc`. Os valores vêm separados por caractere nulo, então troque por quebra de linha:

```bash
cat /proc/1234/environ | tr '\0' '\n'
```

Você só consegue ler o ambiente de processos do seu próprio usuário (ou de qualquer um, se for root). Senhas e tokens guardados em variáveis ficam, portanto, visíveis para quem tem acesso a esse usuário.

## Processos, serviços e pacotes

```bash
ps aux | grep ssh          # procura um processo
ps -eo pid,user,cmd        # PID, dono e comando de todos os processos
kill 1234                  # encerra o processo de PID 1234
systemctl status ssh       # estado de um serviço
journalctl -u ssh -n 20    # últimas linhas do log do serviço
sudo apt update && sudo apt install nmap   # instala um programa (Debian/Ubuntu)
```

Para ver **quais portas estão abertas e quem as abriu**:

```bash
ss -tulpn
```

Em cada linha, olhe o endereço local: `0.0.0.0:7070` ou `*:7070` aceita conexões de qualquer lugar, enquanto `127.0.0.1:7070` só aceita da própria máquina. A coluna final mostra o processo dono da porta (para ver processos de outros usuários, rode com `sudo`).

## Tarefas agendadas (cron)

O **cron** executa comandos em horários marcados. Cada linha tem cinco campos de tempo e depois o comando:

```text
# minuto hora dia-do-mês mês dia-da-semana  comando
0 17 * * *   /usr/local/bin/backup.sh       # todo dia às 17:00
*/15 * * * * /opt/limpa.sh                  # a cada 15 minutos
```

Onde procurar:

```bash
crontab -l             # as tarefas do seu usuário
ls /etc/cron.d/        # tarefas do sistema, em arquivos
cat /etc/crontab       # tabela geral do sistema
```

Ao achar uma tarefa, leia o **script** que ela executa (`cat`) e entenda o que ele faz, em que horário e com qual usuário.

## Um script em bash

```bash
#!/bin/bash
for i in 1 2 3; do
  echo "Tentativa $i"
done
```

Salve como `teste.sh`, dê `chmod +x teste.sh` e rode com `./teste.sh`.

Para ler um script de outra pessoa, vá linha por linha. Um exemplo comum de backup:

```bash
#!/bin/bash
DATA=$(date +%F)                                  # data de hoje: 2026-05-21
tar czf /backup/portal-$DATA.tgz /opt/portal 2>/dev/null || true   # compacta a pasta
echo "$DATA backup concluído" >> /var/log/backup.log               # registra no log
```

`$(comando)` guarda a saída de um comando, `tar czf` cria um arquivo compactado, `2>/dev/null` esconde os erros e `|| true` faz o script continuar mesmo que o `tar` falhe.

## Docker e containers

O Docker roda programas em containers (veja o conceito na página de sistemas operacionais). Os comandos que você mais vai usar:

```bash
docker run --rm -it ubuntu bash      # abre um container Ubuntu interativo; --rm apaga ao sair
docker run -d -p 8080:80 nginx       # roda em segundo plano e liga a porta 8080 do host à 80 do container
docker ps                            # containers em execução (-a mostra também os parados)
docker images                        # imagens baixadas
docker logs nome                     # logs do container
docker exec -it nome bash            # abre um shell dentro de um container que já está rodando
docker stop nome                     # para o container
docker rm nome                       # apaga o container
```

Para criar a sua própria imagem, escreva um `Dockerfile`:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY app.py .
CMD ["python3", "app.py"]
```

```bash
docker build -t meu-app .      # constrói a imagem
docker run --rm meu-app        # executa
```

Dois cuidados: dados dentro do container somem quando ele é apagado (use `-v pasta:/pasta` para guardar fora), e **estar no grupo `docker` equivale a ter acesso de root** na máquina, então não dê esse acesso a qualquer um.

## Lendo um servidor desconhecido

Em CTFs, laboratórios e auditorias, a rotina é sempre parecida. Um roteiro para começar:

1. **Quem sou eu?** `whoami`, `id`, `sudo -l`.
2. **Onde estou?** `cat /etc/os-release`, `hostname`, `uname -a`.
3. **Quem mais existe?** `cat /etc/passwd`, `getent group sudo`.
4. **O que está rodando?** `ps aux`, `ss -tulpn`, `systemctl list-units --type=service --state=running`.
5. **O que está agendado?** `crontab -l`, `ls /etc/cron.d`.
6. **O que há de estranho nos arquivos?** `ls -la ~`, SUID com `find`, configurações em `/etc` e `/opt`, arquivos recentes.
7. **Há segredos expostos?** `grep -r` por `password`, `senha`, `token` e `key`; `env`; `cat ~/.bash_history`; `/proc/<PID>/environ` dos processos.

Quando achar um segredo, **não pare nele**: o que ele abre? Uma chave de API aponta para um serviço, e uma senha pode abrir um painel que você viu escutando em uma porta. Anote o caminho que você percorreu, porque explicar o raciocínio vale tanto quanto achar a resposta.

O que costuma ser **risco** em um servidor, e por quê:

- Credenciais em texto puro, em arquivos de configuração, no histórico de comandos ou em variáveis de ambiente: quem lê o arquivo, ganha a senha.
- Logs e backups com dados sensíveis e permissão aberta para todos (`644`).
- Binários SUID customizados: uma falha neles vira acesso de root.
- Usuários com `sudo` sem restrição: uma conta comprometida compromete a máquina inteira.
- Serviços de teste (como `python3 -m http.server`) escutando em `0.0.0.0`, às vezes listando diretórios inteiros.

E o que fazer sobre cada um: remover ou rotacionar os segredos, apertar as permissões (`chmod 600` para arquivos sensíveis), remover o SUID que não for necessário, restringir o `sudo` aos comandos de que a pessoa precisa e trocar serviços de teste por um servidor de verdade, com autenticação e firewall.

## Mão na massa

1. Crie uma pasta, entre nela e crie três arquivos. Liste com `ls -la`.
2. Tire a permissão de leitura de um dos arquivos e tente ler. Leia a mensagem de erro.
3. Conte quantos usuários existem em `/etc/passwd` usando um pipe. Depois conte só os que têm UID a partir de 1000.
4. Descubra quem tem `sudo` na sua máquina e a qual grupo o seu usuário pertence.
5. Use `find` para listar os binários com SUID do seu sistema. Quais você reconhece?
6. Crie um script que grava a data em um arquivo de log, agende-o no `crontab -e` para rodar a cada minuto e confira se o log cresce.
7. Rode `python3 -m http.server 8000` e, em outro terminal, use `ss -tulpn` para ver em qual endereço ele escuta. Depois use `curl` nele. Qual é o risco de deixá-lo assim?
8. Exporte uma variável com um "segredo" e leia o valor dela em `/proc/<PID>/environ` de um processo seu.
9. Rode um container com `docker run --rm -it ubuntu bash`, instale algo com `apt` e saia. O que acontece com o que você instalou?
10. Jogue os primeiros níveis do **Bandit**, do OverTheWire. É o melhor treino para o terminal.

## Para aprofundar

- [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/): jogo para aprender Linux.
- [Linux Journey](https://linuxjourney.com/): curso gratuito, passo a passo.
- [Guia Foca Linux](https://guiafoca.org/guiaonline/): guia completo, em português.
- [Docker: Getting Started](https://docs.docker.com/get-started/): o tutorial oficial.
- [The Linux Command Line](https://linuxcommand.org/tlcl.php): livro gratuito.
