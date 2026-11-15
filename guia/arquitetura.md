---
ordem: 4
title: "Arquitetura de computadores"
description: "Arquitetura de computadores para iniciantes: CPU, memória, binário e assembly."
---

Para entender engenharia reversa, exploração de binários e malware, você precisa saber o que acontece dentro da máquina. Aqui vai o essencial.

## Números: binário e hexadecimal

O computador só entende bits (0 e 1). Oito bits formam um **byte**. Como o binário é longo, usamos o **hexadecimal** (base 16, com os dígitos de 0 a 9 e de A a F): `0xFF` vale 255. Dois dígitos hexadecimais cabem exatamente em um byte.

Alguns valores que aparecem muito: a letra `A` é `0x41` na tabela ASCII, e `0x00` marca o fim de uma string em C.

**Endianness:** é a ordem em que os bytes de um número ficam na memória. Nos processadores x86, o byte menos significativo vem primeiro (*little-endian*). Assim, `0x12345678` é guardado como `78 56 34 12`.

```bash
printf '%d\n' 0xff         # converte hexa para decimal
echo -n "A" | xxd          # mostra os bytes de um texto
```

## A CPU

A CPU executa instruções em um ciclo: **busca**, **decodifica**, **executa**. Ela tem poucos "cofres" internos muito rápidos chamados **registradores**. No x86-64, os principais são:

| Registrador | Papel comum |
|---|---|
| `rax` | Resultado de operações e retorno de funções |
| `rbx`, `rcx`, `rdx` | Uso geral |
| `rsi`, `rdi` | Argumentos de funções |
| `rsp` | Aponta para o topo da pilha |
| `rbp` | Base do quadro da pilha |
| `rip` | Endereço da próxima instrução |

## A memória

Há uma hierarquia: os registradores são os mais rápidos e pequenos, depois vêm o **cache** (L1, L2, L3), a **RAM** e, por fim, o disco. Dois espaços da RAM importam muito:

- **Stack (pilha):** guarda variáveis locais e o endereço de retorno das funções. Cresce em direção a endereços menores.
- **Heap:** memória alocada dinamicamente, por exemplo com `malloc`.

## Do código ao executável

O código em C passa por etapas até virar um programa:

**código-fonte → compilador → assembly → montador → objeto → linker → executável**

No Linux o formato do executável é o **ELF**, e no Windows é o **PE**. Veja isso na prática:

```bash
gcc -o hello hello.c       # compila
file hello                 # mostra o tipo do arquivo
strings hello | head       # textos legíveis dentro do binário
readelf -h hello           # cabeçalho do ELF
objdump -d -M intel hello | less   # desmonta em assembly
```

## Assembly x86-64 em cinco ideias

Um trecho em sintaxe Intel:

```asm
mov rax, 5      ; coloca 5 em rax
add rax, 3      ; rax agora é 8
push rax        ; empilha o valor
pop rbx         ; desempilha para rbx
cmp rax, 8      ; compara rax com 8
je  igual       ; pula para "igual" se forem iguais
```

Com `mov`, `add`/`sub`, `push`/`pop`, `cmp` com saltos (`jmp`, `je`, `jne`) e `call`/`ret`, você já lê boa parte de um código simples.

## Como uma função funciona

Quando o programa chama uma função com `call`, o endereço de retorno é guardado na pilha, e o `ret` o usa para voltar. É por isso que existem os *buffer overflows*: se um programa copia dados demais para uma variável local, eles podem passar por cima de coisas importantes da pilha, como esse endereço de retorno. Entender essa ideia é o primeiro passo para a categoria *pwn* dos CTFs. As proteções do sistema operacional (NX, ASLR, canários) existem justamente para dificultar isso.

## Mão na massa

1. Converta à mão `0x2F`, `0xFF` e o número 100 entre decimal e hexadecimal. Confira com `printf`.
2. Compile este programa e rode duas vezes. O endereço muda? Por quê?

```c
#include <stdio.h>

int main(void) {
    int x = 42;
    printf("endereço de x: %p\n", (void *)&x);
    return 0;
}
```

3. Use `objdump -d -M intel` no binário e ache a função `main`.
4. Abra o binário no **Ghidra** e compare com o que o `objdump` mostrou.

## Para aprofundar

- [Nand2Tetris](https://www.nand2tetris.org/): construa um computador do zero.
- [CS50](https://cs50.harvard.edu/x/): introdução à computação, com C.
- [pwn.college](https://pwn.college/): trilha gratuita de exploração e arquitetura.
