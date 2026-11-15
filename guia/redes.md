---
ordem: 1
title: "Redes de computadores"
description: "Redes de computadores para iniciantes: camadas, IP, portas, TCP, DNS e HTTP."
---

Quase tudo em segurança passa pela rede: um ataque chega por ela, uma defesa a observa. Entender como ela funciona é a base de todo o resto.

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

| Porta | Serviço |
|---|---|
| 22 | SSH |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 21 | FTP |
| 25 | SMTP |
| 445 | SMB |
| 3306 | MySQL |
| 3389 | RDP |

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
- **HTTP:** o protocolo da web, com métodos (GET, POST), cabeçalhos e códigos de status (200, 404, 500).

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

## Para aprofundar

- [Cisco Networking Academy](https://www.netacad.com/): cursos gratuitos de redes.
- [Professor Messer](https://www.professormesser.com/): vídeos gratuitos de redes.
- [Wizard Zines](https://wizardzines.com/): zines ilustrados sobre DNS, HTTP e redes.
- Livro *Redes de Computadores e a Internet*, de Kurose e Ross.
