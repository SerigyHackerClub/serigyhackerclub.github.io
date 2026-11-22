---
ordem: 1
title: "Redes de computadores"
description: "Redes de computadores para iniciantes: camadas, IP, portas, TCP, DNS, HTTP e análise de tráfego com Wireshark."
---

Quase tudo em segurança passa pela rede: um ataque chega por ela, uma defesa a observa. Entender como ela funciona é a base de todo o resto.

## O que é uma rede

Uma rede de computadores é um conjunto de dispositivos ligados para trocar dados. Quase tudo nela segue o modelo **cliente e servidor**: o cliente (o seu navegador, o `curl`, um app) faz um **pedido**, e o servidor (um site, um serviço de e-mail, um DNS) devolve uma **resposta**. Um mesmo computador pode ser cliente de um serviço e servidor de outro.

## O modelo em camadas

A comunicação é dividida em camadas, e cada uma cuida de uma parte do problema. O modelo usado na internet é o TCP/IP, com quatro camadas:

| Camada | O que faz | Exemplos |
|---|---|---|
| Aplicação | O serviço que você usa | HTTP, DNS, SSH |
| Transporte | Entrega entre programas, por portas | TCP, UDP |
| Internet | Endereçamento e rota entre redes | IP, ICMP |
| Enlace | Comunicação na rede local | Ethernet, Wi-Fi, ARP |

Você vai ouvir falar também do modelo OSI, de sete camadas. Ele é mais usado como referência para estudo, e vale conhecer os dois nomes.

## Endereços

- **MAC:** identifica a placa de rede dentro da rede local. Tem 48 bits, por exemplo `a4:5e:60:d1:22:9b`.
- **IP:** identifica a máquina na rede. O IPv4 tem 32 bits (`192.168.1.68`) e o IPv6 tem 128.
- **Máscara e CIDR:** dizem quantos bits do IP são da rede e quantos são da máquina. `192.168.1.0/24` significa que 24 bits são da rede, então sobram 8 bits para máquinas: 256 endereços, dos quais 254 são usáveis.
- **Faixas privadas:** `10.0.0.0/8`, `172.16.0.0/12` e `192.168.0.0/16` não são roteadas na internet. É por isso que a rede da sua casa usa uma delas.
- **Gateway:** o roteador que leva o seu tráfego para fora da rede local.
- **NAT:** permite que várias máquinas com IP privado saiam para a internet com um único IP público.

## Portas e protocolos

O IP leva o pacote até a máquina, e a **porta** (de 0 a 65535) diz qual programa deve recebê-lo.

| Porta | Serviço | Criptografado? |
|---|---|---|
| 21 | FTP | Não |
| 22 | SSH | Sim |
| 23 | Telnet | Não |
| 25 | SMTP (envio de e-mail) | Em geral não |
| 53 | DNS | Em geral não |
| 80 | HTTP | Não |
| 88 | Kerberos (autenticação no Windows) | Parcialmente |
| 110 | POP3 (leitura de e-mail) | Não |
| 143 | IMAP (leitura de e-mail) | Não |
| 389 | LDAP (diretório de usuários) | Não |
| 443 | HTTPS | Sim |
| 445 | SMB (compartilhamento de arquivos) | Depende da versão |
| 3306 | MySQL | Depende da configuração |
| 3389 | RDP | Sim |

**TCP** abre uma conexão antes de enviar dados (o *handshake* de três passos: SYN, SYN-ACK, ACK) e garante que tudo chegue, em ordem. **UDP** não abre conexão e não garante entrega, mas é mais leve. DNS e streaming usam muito UDP.

## O que acontece quando você abre um site

1. O **DNS** converte o nome (`exemplo.com`) em um endereço IP.
2. O seu computador abre uma conexão **TCP** com esse IP, na porta 443.
3. O **TLS** negocia a criptografia da conexão (é o "S" do HTTPS).
4. O navegador envia um pedido **HTTP**, e o servidor responde com a página.

## Peças que aparecem o tempo todo

- **DNS:** a "lista telefônica" que converte nomes em IPs.
- **DHCP:** entrega IP, máscara e gateway automaticamente quando você conecta.
- **ARP:** descobre qual MAC tem determinado IP na rede local.
- **ICMP:** mensagens de controle, como o `ping`.
- **HTTP:** o protocolo da web, descrito a seguir.

## HTTP em detalhe

Toda conversa HTTP é um **pedido** seguido de uma **resposta**, em texto puro:

```http
POST /login HTTP/1.1
Host: exemplo.com
User-Agent: curl/8.5.0
Content-Type: application/x-www-form-urlencoded

user=ana&senha=1234
```

```http
HTTP/1.1 200 OK
Content-Type: text/html

<html>...</html>
```

- **Método:** o que o cliente quer fazer. `GET` pede um recurso, `POST` envia dados, e há outros como `PUT` e `DELETE`.
- **Caminho (path):** o recurso pedido, como `/login` ou `/vagas`. Depois do `?` vêm os parâmetros: `/busca?q=ana`.
- **Cabeçalhos:** metadados do pedido e da resposta. Alguns que você vai ver sempre: `Host` (qual site), `User-Agent` (qual programa está pedindo: navegador, `curl`, um script em Python), `Content-Type`, `Authorization` e `Cookie`.
- **Corpo:** os dados enviados no `POST` ou o conteúdo da resposta.
- **Códigos de status:** `200` deu certo, `301`/`302` redirecionamento, `401` precisa autenticar, `403` proibido, `404` não existe, `405` método não permitido, `500` erro no servidor.

**HTTP × HTTPS:** o HTTP viaja em texto puro. Qualquer pessoa que consiga ver o tráfego (na mesma rede Wi-Fi, por exemplo) lê tudo: caminhos, parâmetros, senhas e tokens. O HTTPS coloca TLS no meio e esconde o conteúdo. Repare que dados em um `GET` ficam na própria URL, o que é ainda pior, porque aparecem em logs e no histórico.

Atenção com a autenticação "Basic" do HTTP: ela só codifica `usuario:senha` em base64, e isso **não é criptografia**. Qualquer um decodifica.

### Falando com um servidor pelo terminal

O `curl` deixa você montar o pedido na mão. Uma boa estratégia é começar simples e **ler a mensagem de erro**, porque muitos serviços dizem exatamente o que está faltando:

```bash
curl https://api.exemplo.com/validate
# erro: método não permitido, use POST

curl -X POST https://api.exemplo.com/validate
# erro: falta o cabeçalho X-Algo

curl -X POST https://api.exemplo.com/validate \
  -H 'Content-Type: application/json' \
  -H 'X-Algo: valor' \
  -d '{"token":"abc123"}'
```

Outras opções úteis: `-v` (mostra o pedido e a resposta completos), `-i` (mostra só os cabeçalhos da resposta), `-u usuario:senha` (autenticação Basic) e `-A "nome"` (troca o User-Agent).

## Protocolos sem criptografia

Telnet, FTP, HTTP, POP3, IMAP e LDAP simples mandam usuário e senha **em texto puro**. Quem captura o tráfego lê as credenciais sem nenhum esforço. É por isso que o SSH substituiu o Telnet e o HTTPS substituiu o HTTP. Ao analisar uma captura, esses são os primeiros protocolos a procurar.

## Analisando tráfego

Uma **captura de pacotes** (arquivo `.pcap` ou `.pcapng`) é a gravação do que passou pela rede. O **Wireshark** abre esses arquivos com interface gráfica, e o **tshark** faz o mesmo no terminal.

### Filtros de exibição do Wireshark

Em uma captura grande, o filtro é a ferramenta principal:

| Filtro | O que mostra |
|---|---|
| `dns` | Só consultas e respostas DNS |
| `http` | Só HTTP |
| `http.request` | Só os pedidos HTTP |
| `http.request.uri contains "login"` | Pedidos cujo caminho tem "login" |
| `http.user_agent contains "python"` | Pedidos feitos por um User-Agent específico |
| `ip.addr == 10.0.0.5` | Tudo que envolve esse IP |
| `ip.src == 10.0.0.5` | Só o que esse IP enviou |
| `tcp.port == 23` | Tráfego na porta 23 |
| `icmp` | Pings |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Tentativas de abrir conexão |
| `frame contains "senha"` | Qualquer pacote com esse texto |

Dá para combinar com `&&` (e), `||` (ou) e `!` (não). Outros atalhos valiosos:

- **Follow TCP Stream** (botão direito em um pacote): reconstrói a conversa inteira em texto. É a forma mais rápida de ler um login em Telnet ou HTTP.
- **Statistics → Conversations** e **Protocol Hierarchy**: mostram quem falou com quem e quais protocolos aparecem, ótimo para ter uma visão geral.
- **File → Export Objects → HTTP**: extrai os arquivos que foram baixados na captura.

### O mesmo no terminal com tshark

```bash
tshark -r captura.pcap -Y 'dns' -T fields -e ip.src -e dns.qry.name
tshark -r captura.pcap -Y 'http.request' -T fields -e ip.src -e http.host -e http.request.uri -e http.user_agent
tshark -r captura.pcap -q -z conv,tcp
```

Vale conferir a análise manual com o tshark, porque ele não "se cansa" e ajuda a achar o que passou batido.

### Conexão real ou só uma tentativa?

Uma pergunta comum em análise de captura: um protocolo "apareceu" ou foi usado de verdade? Um bom critério é olhar se houve:

1. o **handshake TCP completo** (SYN, SYN-ACK, ACK), e não só um SYN sem resposta ou um `RST`;
2. **troca de dados** da camada de aplicação, com resposta do servidor.

Um SYN seguido de `RST` significa que a porta estava fechada ou que a conexão foi recusada. Isso é uma tentativa, não uma comunicação.

### Exfiltração de dados por DNS

O DNS quase nunca é bloqueado, e por isso é usado para esconder vazamento de dados. O atacante codifica a informação (em base32 ou base64) e a envia aos poucos, como **subdomínios** de um domínio que ele controla: `MFRGG2.telemetria-x.net`, `ZLTOON.telemetria-x.net` e assim por diante. Sinais de alerta:

- muitas consultas para o mesmo domínio incomum;
- subdomínios longos e com aparência aleatória;
- consultas do tipo `TXT` ou volume de DNS bem acima do normal para aquele host.

Para investigar: filtre o DNS, ache o domínio suspeito, extraia os subdomínios **na ordem em que apareceram**, junte tudo e tente decodificar:

```bash
echo "MFRGG2LTOQ======" | base32 -d
echo "dGVzdGU=" | base64 -d
```

## Mão na massa

No Linux, no terminal (use só na sua própria rede):

```bash
ip a                      # seus endereços de rede
ip route                  # rota padrão (gateway)
ping -c 4 1.1.1.1         # testa se há conexão
dig exemplo.com           # consulta DNS
curl -v https://example.com   # veja o handshake TLS e os cabeçalhos HTTP
ss -tulpn                 # portas abertas na sua máquina e quem as abriu
sudo tcpdump -i any -n port 53   # veja consultas DNS passando
```

Exercícios:

1. Descubra o seu IP, a máscara e o gateway. Calcule quantas máquinas cabem na sua rede.
2. Abra o **Wireshark**, filtre por `dns` e visite um site. Encontre a consulta e a resposta.
3. Rode o `curl -v` em um site e identifique o handshake, a versão do TLS e os cabeçalhos da resposta.
4. Com `ss -tulpn`, liste os serviços escutando na sua máquina. Você reconhece todos?
5. Capture o tráfego com o Wireshark enquanto roda `curl -i http://example.com` (HTTP, sem o "S"). Use **Follow TCP Stream** e leia o pedido e a resposta em texto puro.
6. Baixe uma captura de treino (veja os links abaixo). Descubra qual IP mais fez consultas DNS, quais User-Agents aparecem e se há algum protocolo sem criptografia com credenciais visíveis.
7. Repita a análise da questão anterior com o `tshark`, sem abrir a interface gráfica.
8. Use `curl` para enviar um `POST` com um cabeçalho personalizado para o `https://httpbin.org/post` e compare o que você enviou com o que ele devolve.

## Para aprofundar

- [Cisco Networking Academy: Networking Basics](https://www.netacad.com/pt/courses/networking-basics?courseLang=pt-BR): curso gratuito de redes, em português.
- [Playlist de redes de computadores (YouTube)](https://www.youtube.com/playlist?list=PLxI8Can9yAHc-_dZ6nsfoon08i2-4OvEk): aulas em português.
- [Gustavo Kalau (YouTube)](https://www.youtube.com/@GustavoKalau/courses): cursos em português.
- [MDN: HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP): a melhor referência sobre HTTP, com métodos, cabeçalhos e status.
- [MDN: Primeiro site](https://developer.mozilla.org/pt-BR/docs/Learn_web_development/Getting_started/Your_first_website): útil para entender o lado do servidor e do navegador.
- [Wireshark: materiais de aprendizado](https://www.wireshark.org/learn): documentação oficial e vídeos.
- [Como usar o Wireshark (Varonis)](https://www.varonis.com/blog/how-to-use-wireshark): introdução prática.
- [Malware Traffic Analysis](https://www.malware-traffic-analysis.net/): capturas reais para treinar. Analise-as sempre dentro de uma VM.
- [Professor Messer](https://www.professormesser.com/): vídeos gratuitos de redes.
- [Wizard Zines](https://wizardzines.com/): zines ilustrados sobre DNS, HTTP e redes.
- Livro *Redes de Computadores e a Internet*, de Kurose e Ross.
