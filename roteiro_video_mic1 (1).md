# Roteiro integral do vídeo — IJVM no simulador MIC1MMV

**Disciplina:** Sistemas Microprocessados — UFBA  
**Autor:** `[SEU NOME COMPLETO]` (substituir antes de gravar)  
**Dados usados:** último dígito do CPF `a=2`; último dígito da matrícula `b=5`.  
**Código:** `mini_projeto_mic1_ijvm.jas` com **dois `ILOAD a`** (sem `DUP`).  
**Objetivo:** demonstrar compilação, IJVM, microinstruções e registradores em nível de clock.

> **Limite de precisão:** este roteiro deriva do microprograma de referência `mic1ijvm.mal` (MIC-1 clássica). A numeração dos clocks abaixo **começa no `Main1` associado ao primeiro `BIPUSH 2`**, e não necessariamente no clock 0 do simulador. O simulador pode executar ciclos iniciais de `NOP`/busca antes disso, e uma variante de microcódigo pode alterar a sequência. Confirme, na sua tela, cada microinstrução antes de narrar um valor de registrador. O valor de MPC, os endereços absolutos e o instante em que o MBR recebe um byte podem variar conforme a interface e a versão. Não invente esses valores.

## 1. Preparação da gravação

1. Abra o `mini_projeto_mic1_ijvm.jas` e o MIC1MMV. Mantenha seu nome completo, disciplina e identificação de autor visíveis em sobreposição durante a gravação inteira.
2. Importe o arquivo em `File > Assemble / Load JAS File...`. Mostre `assembly complete` antes de clicar em `Load`.
3. Abra a visão de registradores e a visualização de memória/pilha, quando presentes. Deixe visíveis `PC`, `MPC`, `MBR`, `MAR`, `MDR`, `SP`, `LV`, `TOS`, `H`, `OPC` e flags de condição, se a interface exibir.
4. Primeiro valide o resultado no modo `IJVM`; depois reinicie a execução, selecione `Clock` e siga o roteiro detalhado abaixo, um clique por linha.
5. Se aparecerem microinstruções iniciais antes do primeiro `Main1` que despacha `BIPUSH 2`, descreva-as com a explicação da Seção 4.1 e **comece a contagem relativa** apenas depois de identificar a sequência do primeiro `BIPUSH`.

### Fala de abertura (ler/adaptar)

> "Neste mini projeto, vou traduzir um programa de alto nível para IJVM e verificar seu funcionamento na microarquitetura MIC-1. Minha variável a vale 2 e a variável b vale 5. Primeiro calculo duas vezes a menos b, resultando em menos 1. Como menos 1 não é maior que 3, o programa deve executar o else, somar 4 e terminar com c igual a 3. Vou mostrar as instruções IJVM e, depois, cada microinstrução em nível de clock."

## 2. Lógica e código completo

```text
a = 2;
b = 5;
c = 2*a - b;  // c = 4 - 5 = -1
if (c > 3) c = c - 4;
else       c = c + 4;   // resultado: 3
```

A IJVM não possui `IMUL` na implementação básica. Portanto, usamos `ILOAD a`, `ILOAD a`, `IADD`, somando duas cópias de `a`. Para testar `c > 3`, fazemos `3 - c` e usamos `IFLT`: se o resultado for negativo, `c` era maior que 3.

```asm
// Mini projeto - Sistemas Microprocessados (UFBA)
// a = último dígito do CPF = 2
// b = último dígito da matrícula = 5
// c = 2*a - b
// if (c > 3) c = c - 4; else c = c + 4;
// Usa dois ILOAD a, conforme a abordagem vista em aula.

.main

.var
a
b
c
.end-var

    // a = 2
    BIPUSH 2
    ISTORE a

    // b = 5
    BIPUSH 5
    ISTORE b

    // c = a + a - b
    ILOAD a
    ILOAD a
    IADD
    ILOAD b
    ISUB
    ISTORE c

    // c > 3 equivale a 3 - c < 0
    BIPUSH 3
    ILOAD c
    ISUB
    IFLT MAIOR

    // ELSE: c = c + 4
    ILOAD c
    BIPUSH 4
    IADD
    ISTORE c
    GOTO FIM

MAIOR:
    // IF: c = c - 4 (não executado neste caso)
    ILOAD c
    BIPUSH 4
    ISUB
    ISTORE c

FIM:
    // Exibe 3 no topo da pilha e encerra
    ILOAD c
    HALT

.end-main
```

### Fala sobre a tradução

> "A IJVM trabalha com uma pilha. BIPUSH empilha uma constante, ILOAD carrega uma variável local, ISTORE grava uma variável, IADD soma os dois elementos do topo e ISUB subtrai o elemento superior do imediatamente abaixo. Usei ILOAD a duas vezes para implementar a multiplicacao por dois. Na condição, testo 3 menos c com IFLT; se o valor for negativo, salto para MAIOR. Caso contrario, continuo pelo else."

## 3. Validação previa no modo IJVM

| Ordem | Instrução executada | Estado logico após a instrução |
|---:|---|---|
| 01 | `BIPUSH 2` | Pilha lógica: [2]. |
| 02 | `ISTORE a` | Variáveis: a=2; pilha lógica vazia. |
| 03 | `BIPUSH 5` | Pilha lógica: [5]. |
| 04 | `ISTORE b` | Variáveis: a=2, b=5; pilha lógica vazia. |
| 05 | `ILOAD a (primeiro)` | Pilha lógica: [2]. |
| 06 | `ILOAD a (segundo)` | Pilha lógica: [2, 2]. |
| 07 | `IADD (2 + 2)` | Pilha lógica: [4]; TOS=4. |
| 08 | `ILOAD b` | Pilha lógica: [4, 5]; TOS=5. |
| 09 | `ISUB (4 - 5)` | Pilha lógica: [-1]; TOS=-1 (0xFFFFFFFF em 32 bits). |
| 10 | `ISTORE c (primeiro)` | Variáveis: a=2, b=5, c=-1; pilha lógica vazia. |
| 11 | `BIPUSH 3` | Pilha lógica: [3]. |
| 12 | `ILOAD c (comparação)` | Pilha lógica: [3, -1]. |
| 13 | `ISUB (3 - (-1))` | Pilha lógica: [4]; TOS=4. |
| 14 | `IFLT MAIOR (não tomado)` | O caminho escolhido é o ELSE. O valor 4 da comparação foi desempilhado. |
| 15 | `ILOAD c (else)` | Pilha lógica: [-1]. |
| 16 | `BIPUSH 4 (else)` | Pilha lógica: [-1, 4]. |
| 17 | `IADD (-1 + 4)` | Pilha lógica: [3]; TOS=3. |
| 18 | `ISTORE c (segundo)` | Variáveis: a=2, b=5, c=3; pilha lógica vazia. |
| 19 | `GOTO FIM` | PC aponta para o destino FIM (ILOAD c); o bloco MAIOR foi ignorado. |
| 20 | `ILOAD c (resultado)` | Pilha lógica: [3]; TOS=3 (0x00000003). |
| 21 | `HALT` | Programa encerrado; c=3 e TOS=3. |

> **Importante:** a pilha lógica acima representa os operandos do algoritmo, não necessariamente todas as palavras exibidas na RAM. A área de variáveis locais e a pilha de operandos são regioes diferentes; TOS pode apresentar um valor anterior quando a pilha lógica está vazia.

## 4. Execução em nível de clock — roteiro exaustivo

### 4.1 Inicialização e sincronizacao da contagem

Ao clicar em `Reset` e selecionar `Clock`, a máquina pode executar antes do seu código uma sequência de inicialização com `nop1` e `Main1`. Não some esses cliques aos números abaixo. O próprio simulador pode exibir a microinstrução recém-executada em vez da próxima, então observe os efeitos nos registradores para distinguir as duas situações.

**Falas para a inicialização (quando aparecer):**

- `nop1`: "A microinstrução NOP não altera os operandos; ela retorna para o ciclo principal de busca."
- `Main1`: "Este é o ciclo de busca e despacho: o PC avança, um fetch é iniciado, e o próximo endereço de microcódigo é escolhido a partir do opcode presente no MBR."
- `BIPUSH`: "Agora o programa chegou a primeira instrução do meu calculo: empilhar a constante 2."

**Como usar as tabelas:** cada linha representa **um clique em modo Clock**, seguindo o microprograma de referência. Leia a coluna **"Fala curta"** enquanto aponta para os registradores da coluna **"O que conferir"**. Você pode narrar em ritmo natural e pausar entre os cliques.

### 4.2 Instrução IJVM 01 — `BIPUSH 2` (clocks relativos 001–004)

**Objetivo:** Inicializar a com 2: empilhar a constante.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **001** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **002** | `bipush1` — `SP = MAR = SP + 1` | Reserva uma nova palavra na pilha: SP e MAR aumentam em uma unidade. | "Primeiro, incremento SP e MAR para criar uma posição na pilha." |
| **003** | `bipush2` — `PC = PC + 1; fetch` | PC avança e inicia a busca do próximo opcode. | "Agora o PC avança e antecipa a busca da próxima instrução IJVM." |
| **004** | `bipush3` — `MDR = TOS = MBR; wr; goto Main1` | MBR tem o imediato 2; MDR e TOS recebem 2; inicia escrita na pilha. | "O valor imediato 2 passa do MBR para MDR e TOS, e a memória recebe a escrita." |

**Checkpoint após a instrução:** Pilha lógica: [2].

### 4.3 Instrução IJVM 02 — `ISTORE a` (clocks relativos 005–011)

**Objetivo:** Armazenar 2 na variável a.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **005** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **006** | `istore1` — `H = LV` | H copia a base do quadro de variáveis locais (LV). | "Primeiro guardo o endereço-base LV no auxiliar H." |
| **007** | `istore2` — `MAR = MBRU + H` | O índice de a em MBRU soma-se a LV, formando o endereço da variável em MAR. | "Somo o índice de a ao LV para endereçar a variável local." |
| **008** | `istore3` — `MDR = TOS; wr` | O valor do topo (2) é enviado a MDR e gravado na local a. | "Aqui o valor 2 do topo é armazenado na variável a." |
| **009** | `istore4` — `SP = MAR = SP - 1; rd` | A pilha perde um elemento; inicia leitura do novo topo. | "Depois do armazenamento, removo o elemento utilizado e solicito a leitura do novo topo." |
| **010** | `istore5` — `PC = PC + 1; fetch` | Avança ao próximo bytecode e inicia sua busca. | "Avanço o PC para preparar a próxima instrução IJVM." |
| **011** | `istore6` — `TOS = MDR; goto Main1` | Restaura TOS com a palavra remanescente no topo; retorna a Main1. | "Concluo o desempilhamento atualizando TOS e retorno ao despacho." |

**Checkpoint após a instrução:** Variáveis: a=2; pilha lógica vazia.

### 4.4 Instrução IJVM 03 — `BIPUSH 5` (clocks relativos 012–015)

**Objetivo:** Preparar a inicialização de b.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **012** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **013** | `bipush1` — `SP = MAR = SP + 1` | Reserva uma nova palavra na pilha: SP e MAR aumentam em uma unidade. | "Primeiro, incremento SP e MAR para criar uma posição na pilha." |
| **014** | `bipush2` — `PC = PC + 1; fetch` | PC avança e inicia a busca do próximo opcode. | "Agora o PC avança e antecipa a busca da próxima instrução IJVM." |
| **015** | `bipush3` — `MDR = TOS = MBR; wr; goto Main1` | MBR tem o imediato 5; MDR e TOS recebem 5; inicia escrita na pilha. | "O valor imediato 5 passa do MBR para MDR e TOS, e a memória recebe a escrita." |

**Checkpoint após a instrução:** Pilha lógica: [5].

### 4.5 Instrução IJVM 04 — `ISTORE b` (clocks relativos 016–022)

**Objetivo:** Armazenar 5 na variável b.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **016** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **017** | `istore1` — `H = LV` | H copia a base do quadro de variáveis locais (LV). | "Primeiro guardo o endereço-base LV no auxiliar H." |
| **018** | `istore2` — `MAR = MBRU + H` | O índice de b em MBRU soma-se a LV, formando o endereço da variável em MAR. | "Somo o índice de b ao LV para endereçar a variável local." |
| **019** | `istore3` — `MDR = TOS; wr` | O valor do topo (5) é enviado a MDR e gravado na local b. | "Aqui o valor 5 do topo é armazenado na variável b." |
| **020** | `istore4` — `SP = MAR = SP - 1; rd` | A pilha perde um elemento; inicia leitura do novo topo. | "Depois do armazenamento, removo o elemento utilizado e solicito a leitura do novo topo." |
| **021** | `istore5` — `PC = PC + 1; fetch` | Avança ao próximo bytecode e inicia sua busca. | "Avanço o PC para preparar a próxima instrução IJVM." |
| **022** | `istore6` — `TOS = MDR; goto Main1` | Restaura TOS com a palavra remanescente no topo; retorna a Main1. | "Concluo o desempilhamento atualizando TOS e retorno ao despacho." |

**Checkpoint após a instrução:** Variáveis: a=2, b=5; pilha lógica vazia.

### 4.6 Instrução IJVM 05 — `ILOAD a (primeiro)` (clocks relativos 023–028)

**Objetivo:** Buscar a primeira copia de a.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **023** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **024** | `iload1` — `H = LV` | Copia a base das variáveis locais para H. | "Copio a base do quadro de variáveis, LV, para H." |
| **025** | `iload2` — `MAR = MBRU + H; rd` | Calcula o endereço de a e inicia leitura da memória. | "Calculo o endereço de a e solicito sua leitura na memória." |
| **026** | `iload3` — `MAR = SP = SP + 1` | Abre uma nova posição na pilha para o valor carregado. | "Incremento SP e MAR para reservar espaco na pilha." |
| **027** | `iload4` — `PC = PC + 1; fetch; wr` | Busca a próxima instrução e escreve na pilha o valor já lido. | "A leitura da variável já ocorreu; a escrita a coloca na nova posição da pilha." |
| **028** | `iload5` — `TOS = MDR; goto Main1` | Atualiza o topo para 2; retorna a Main1. | "TOS passa a valer 2. Com isso a instrução ILOAD termina." |

**Checkpoint após a instrução:** Pilha lógica: [2].

### 4.7 Instrução IJVM 06 — `ILOAD a (segundo)` (clocks relativos 029–034)

**Objetivo:** Buscar a segunda copia de a; equivalente a preparar a+a.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **029** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **030** | `iload1` — `H = LV` | Copia a base das variáveis locais para H. | "Copio a base do quadro de variáveis, LV, para H." |
| **031** | `iload2` — `MAR = MBRU + H; rd` | Calcula o endereço de a e inicia leitura da memória. | "Calculo o endereço de a e solicito sua leitura na memória." |
| **032** | `iload3` — `MAR = SP = SP + 1` | Abre uma nova posição na pilha para o valor carregado. | "Incremento SP e MAR para reservar espaco na pilha." |
| **033** | `iload4` — `PC = PC + 1; fetch; wr` | Busca a próxima instrução e escreve na pilha o valor já lido. | "A leitura da variável já ocorreu; a escrita a coloca na nova posição da pilha." |
| **034** | `iload5` — `TOS = MDR; goto Main1` | Atualiza o topo para 2; retorna a Main1. | "TOS passa a valer 2. Com isso a instrução ILOAD termina." |

**Checkpoint após a instrução:** Pilha lógica: [2, 2].

**Demonstração importante:** a pilha contém duas cópias independentes do valor `2`. A soma seguinte fara `2 + 2`, sem necessidade de `DUP`.

### 4.8 Instrução IJVM 07 — `IADD (2 + 2)` (clocks relativos 035–038)

**Objetivo:** Calcular o dobro de a.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **035** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **036** | `iadd1` — `MAR = SP = SP - 1; rd` | Reduz a pilha de dois operandos para uma posição e inicia leitura do penultimo operando. | "Decremento SP e solicito a leitura do penultimo elemento da pilha." |
| **037** | `iadd2` — `H = TOS` | Guarda o operando do topo (2) em H. | "Copio o operando que estava no topo para o registrador H." |
| **038** | `iadd3` — `MDR = TOS = MDR + H; wr; goto Main1` | Soma 2 + 2 = 4; atualiza TOS e MDR; grava resultado. | "A ULA soma 2 e 2, produzindo 4. Esse resultado ocupa agora o topo." |

**Checkpoint após a instrução:** Pilha lógica: [4]; TOS=4.

### 4.9 Instrução IJVM 08 — `ILOAD b` (clocks relativos 039–044)

**Objetivo:** Empilhar b para fazer 4 - 5.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **039** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **040** | `iload1` — `H = LV` | Copia a base das variáveis locais para H. | "Copio a base do quadro de variáveis, LV, para H." |
| **041** | `iload2` — `MAR = MBRU + H; rd` | Calcula o endereço de b e inicia leitura da memória. | "Calculo o endereço de b e solicito sua leitura na memória." |
| **042** | `iload3` — `MAR = SP = SP + 1` | Abre uma nova posição na pilha para o valor carregado. | "Incremento SP e MAR para reservar espaco na pilha." |
| **043** | `iload4` — `PC = PC + 1; fetch; wr` | Busca a próxima instrução e escreve na pilha o valor já lido. | "A leitura da variável já ocorreu; a escrita a coloca na nova posição da pilha." |
| **044** | `iload5` — `TOS = MDR; goto Main1` | Atualiza o topo para 5; retorna a Main1. | "TOS passa a valer 5. Com isso a instrução ILOAD termina." |

**Checkpoint após a instrução:** Pilha lógica: [4, 5]; TOS=5.

### 4.10 Instrução IJVM 09 — `ISUB (4 - 5)` (clocks relativos 045–048)

**Objetivo:** Calcular c inicial.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **045** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **046** | `isub1` — `MAR = SP = SP - 1; rd` | Aponta MAR/SP para o penultimo elemento e inicia sua leitura. | "Decremento SP e leio o operando que estava abaixo do topo." |
| **047** | `isub2` — `H = TOS` | Guarda o operando superior (5) em H. | "Guardo em H o operando que será subtraido." |
| **048** | `isub3` — `MDR = TOS = MDR - H; wr; goto Main1` | Subtrai 4 - 5 = -1; registra em MDR/TOS; inicia escrita. | "Faço 4 menos 5, obtendo -1, e atualizo o topo da pilha." |

**Checkpoint após a instrução:** Pilha lógica: [-1]; TOS=-1 (0xFFFFFFFF em 32 bits).

**Ponto para aproximar a tela:** `TOS` em 32 bits deve representar `-1` como `0xFFFFFFFF`. Isso é complemento de dois. O conteúdo da RAM também é atualizado pela rotina `isub3`.

### 4.11 Instrução IJVM 10 — `ISTORE c (primeiro)` (clocks relativos 049–055)

**Objetivo:** Guardar c=-1 na variável local.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **049** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **050** | `istore1` — `H = LV` | H copia a base do quadro de variáveis locais (LV). | "Primeiro guardo o endereço-base LV no auxiliar H." |
| **051** | `istore2` — `MAR = MBRU + H` | O índice de c em MBRU soma-se a LV, formando o endereço da variável em MAR. | "Somo o índice de c ao LV para endereçar a variável local." |
| **052** | `istore3` — `MDR = TOS; wr` | O valor do topo (-1) é enviado a MDR e gravado na local c. | "Aqui o valor -1 do topo é armazenado na variável c." |
| **053** | `istore4` — `SP = MAR = SP - 1; rd` | A pilha perde um elemento; inicia leitura do novo topo. | "Depois do armazenamento, removo o elemento utilizado e solicito a leitura do novo topo." |
| **054** | `istore5` — `PC = PC + 1; fetch` | Avança ao próximo bytecode e inicia sua busca. | "Avanço o PC para preparar a próxima instrução IJVM." |
| **055** | `istore6` — `TOS = MDR; goto Main1` | Restaura TOS com a palavra remanescente no topo; retorna a Main1. | "Concluo o desempilhamento atualizando TOS e retorno ao despacho." |

**Checkpoint após a instrução:** Variáveis: a=2, b=5, c=-1; pilha lógica vazia.

### 4.12 Instrução IJVM 11 — `BIPUSH 3` (clocks relativos 056–059)

**Objetivo:** Empilhar o limite da condição.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **056** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **057** | `bipush1` — `SP = MAR = SP + 1` | Reserva uma nova palavra na pilha: SP e MAR aumentam em uma unidade. | "Primeiro, incremento SP e MAR para criar uma posição na pilha." |
| **058** | `bipush2` — `PC = PC + 1; fetch` | PC avança e inicia a busca do próximo opcode. | "Agora o PC avança e antecipa a busca da próxima instrução IJVM." |
| **059** | `bipush3` — `MDR = TOS = MBR; wr; goto Main1` | MBR tem o imediato 3; MDR e TOS recebem 3; inicia escrita na pilha. | "O valor imediato 3 passa do MBR para MDR e TOS, e a memória recebe a escrita." |

**Checkpoint após a instrução:** Pilha lógica: [3].

### 4.13 Instrução IJVM 12 — `ILOAD c (comparação)` (clocks relativos 060–065)

**Objetivo:** Empilhar c=-1 para calcular 3-c.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **060** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **061** | `iload1` — `H = LV` | Copia a base das variáveis locais para H. | "Copio a base do quadro de variáveis, LV, para H." |
| **062** | `iload2` — `MAR = MBRU + H; rd` | Calcula o endereço de c e inicia leitura da memória. | "Calculo o endereço de c e solicito sua leitura na memória." |
| **063** | `iload3` — `MAR = SP = SP + 1` | Abre uma nova posição na pilha para o valor carregado. | "Incremento SP e MAR para reservar espaco na pilha." |
| **064** | `iload4` — `PC = PC + 1; fetch; wr` | Busca a próxima instrução e escreve na pilha o valor já lido. | "A leitura da variável já ocorreu; a escrita a coloca na nova posição da pilha." |
| **065** | `iload5` — `TOS = MDR; goto Main1` | Atualiza o topo para -1; retorna a Main1. | "TOS passa a valer -1. Com isso a instrução ILOAD termina." |

**Checkpoint após a instrução:** Pilha lógica: [3, -1].

### 4.14 Instrução IJVM 13 — `ISUB (3 - (-1))` (clocks relativos 066–069)

**Objetivo:** Transformar c > 3 no teste 3-c < 0.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **066** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **067** | `isub1` — `MAR = SP = SP - 1; rd` | Aponta MAR/SP para o penultimo elemento e inicia sua leitura. | "Decremento SP e leio o operando que estava abaixo do topo." |
| **068** | `isub2` — `H = TOS` | Guarda o operando superior (-1) em H. | "Guardo em H o operando que será subtraido." |
| **069** | `isub3` — `MDR = TOS = MDR - H; wr; goto Main1` | Subtrai 3 - -1 = 4; registra em MDR/TOS; inicia escrita. | "Faço 3 menos -1, obtendo 4, e atualizo o topo da pilha." |

**Checkpoint após a instrução:** Pilha lógica: [4]; TOS=4.

### 4.15 Instrução IJVM 14 — `IFLT MAIOR (não tomado)` (clocks relativos 070–077)

**Objetivo:** Como 4 não é negativo, executar o else.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **070** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **071** | `iflt1` — `MAR = SP = SP - 1; rd` | Retira a condição da pilha e inicia leitura do novo topo. | "O IFLT consome o valor testado, por isso decrementa o SP." |
| **072** | `iflt2` — `OPC = TOS` | Salva temporariamente o valor testado em OPC: 4. | "Copio para OPC o resultado da comparação, que é 4." |
| **073** | `iflt3` — `TOS = MDR` | Recompoe TOS com o elemento remanescente da pilha. | "Restauro o topo da pilha após retirar a condição." |
| **074** | `iflt4` — `N = OPC; if (N) goto T; else goto F` | OPC=4 tem bit de sinal zero: N=0; escolhe o caminho F, sem salto. | "O valor 4 não é negativo. Portanto, N vale zero e a condição é falsa." |
| **075** | `F` — `PC = PC + 1` | Ignora o primeiro byte do deslocamento do desvio não realizado. | "Como não houve salto, ignoro o primeiro byte do deslocamento." |
| **076** | `F2` — `PC = PC + 1; fetch` | Avança alem do segundo byte do deslocamento e inicia busca do próximo opcode. | "Agora ignoro o segundo byte e busco a instrução seguinte, que pertence ao else." |
| **077** | `F3` — `goto Main1` | Retorna ao despacho normal para prosseguir pelo bloco else. | "Retorno a Main1 e continuo no bloco else." |

**Checkpoint após a instrução:** O caminho escolhido é o ELSE. O valor 4 da comparação foi desempilhado.

**Ponto central do `if/else`:** a flag `N` é zero, então o salto `MAIOR` **não ocorre**. Os passos `F`, `F2` e `F3` descartam os dois bytes do deslocamento e continuam no `else`. Não execute manualmente o bloco `MAIOR`.

### 4.16 Instrução IJVM 15 — `ILOAD c (else)` (clocks relativos 078–083)

**Objetivo:** Empilhar c=-1 para soma de 4.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **078** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **079** | `iload1` — `H = LV` | Copia a base das variáveis locais para H. | "Copio a base do quadro de variáveis, LV, para H." |
| **080** | `iload2` — `MAR = MBRU + H; rd` | Calcula o endereço de c e inicia leitura da memória. | "Calculo o endereço de c e solicito sua leitura na memória." |
| **081** | `iload3` — `MAR = SP = SP + 1` | Abre uma nova posição na pilha para o valor carregado. | "Incremento SP e MAR para reservar espaco na pilha." |
| **082** | `iload4` — `PC = PC + 1; fetch; wr` | Busca a próxima instrução e escreve na pilha o valor já lido. | "A leitura da variável já ocorreu; a escrita a coloca na nova posição da pilha." |
| **083** | `iload5` — `TOS = MDR; goto Main1` | Atualiza o topo para -1; retorna a Main1. | "TOS passa a valer -1. Com isso a instrução ILOAD termina." |

**Checkpoint após a instrução:** Pilha lógica: [-1].

### 4.17 Instrução IJVM 16 — `BIPUSH 4 (else)` (clocks relativos 084–087)

**Objetivo:** Empilhar 4.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **084** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **085** | `bipush1` — `SP = MAR = SP + 1` | Reserva uma nova palavra na pilha: SP e MAR aumentam em uma unidade. | "Primeiro, incremento SP e MAR para criar uma posição na pilha." |
| **086** | `bipush2` — `PC = PC + 1; fetch` | PC avança e inicia a busca do próximo opcode. | "Agora o PC avança e antecipa a busca da próxima instrução IJVM." |
| **087** | `bipush3` — `MDR = TOS = MBR; wr; goto Main1` | MBR tem o imediato 4; MDR e TOS recebem 4; inicia escrita na pilha. | "O valor imediato 4 passa do MBR para MDR e TOS, e a memória recebe a escrita." |

**Checkpoint após a instrução:** Pilha lógica: [-1, 4].

### 4.18 Instrução IJVM 17 — `IADD (-1 + 4)` (clocks relativos 088–091)

**Objetivo:** Obter o resultado final.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **088** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **089** | `iadd1` — `MAR = SP = SP - 1; rd` | Reduz a pilha de dois operandos para uma posição e inicia leitura do penultimo operando. | "Decremento SP e solicito a leitura do penultimo elemento da pilha." |
| **090** | `iadd2` — `H = TOS` | Guarda o operando do topo (4) em H. | "Copio o operando que estava no topo para o registrador H." |
| **091** | `iadd3` — `MDR = TOS = MDR + H; wr; goto Main1` | Soma -1 + 4 = 3; atualiza TOS e MDR; grava resultado. | "A ULA soma -1 e 4, produzindo 3. Esse resultado ocupa agora o topo." |

**Checkpoint após a instrução:** Pilha lógica: [3]; TOS=3.

### 4.19 Instrução IJVM 18 — `ISTORE c (segundo)` (clocks relativos 092–098)

**Objetivo:** Armazenar c=3.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **092** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **093** | `istore1` — `H = LV` | H copia a base do quadro de variáveis locais (LV). | "Primeiro guardo o endereço-base LV no auxiliar H." |
| **094** | `istore2` — `MAR = MBRU + H` | O índice de c em MBRU soma-se a LV, formando o endereço da variável em MAR. | "Somo o índice de c ao LV para endereçar a variável local." |
| **095** | `istore3` — `MDR = TOS; wr` | O valor do topo (3) é enviado a MDR e gravado na local c. | "Aqui o valor 3 do topo é armazenado na variável c." |
| **096** | `istore4` — `SP = MAR = SP - 1; rd` | A pilha perde um elemento; inicia leitura do novo topo. | "Depois do armazenamento, removo o elemento utilizado e solicito a leitura do novo topo." |
| **097** | `istore5` — `PC = PC + 1; fetch` | Avança ao próximo bytecode e inicia sua busca. | "Avanço o PC para preparar a próxima instrução IJVM." |
| **098** | `istore6` — `TOS = MDR; goto Main1` | Restaura TOS com a palavra remanescente no topo; retorna a Main1. | "Concluo o desempilhamento atualizando TOS e retorno ao despacho." |

**Checkpoint após a instrução:** Variáveis: a=2, b=5, c=3; pilha lógica vazia.

### 4.20 Instrução IJVM 19 — `GOTO FIM` (clocks relativos 099–105)

**Objetivo:** Pular o bloco MAIOR, que não pode ser executado.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **099** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **100** | `goto1` — `OPC = PC - 1` | Guarda em OPC o endereço do opcode GOTO. | "Guardo em OPC o endereço da instrução de desvio." |
| **101** | `goto2` — `PC = PC + 1; fetch` | Avança e busca o segundo byte do deslocamento. | "Busco a segunda parte do deslocamento do GOTO." |
| **102** | `goto3` — `H = MBR << 8` | Desloca o primeiro byte do offset para formar a parte alta. | "Desloco o primeiro byte do deslocamento oito bits para a esquerda." |
| **103** | `goto4` — `H = MBRU OR H` | Completa o deslocamento de 16 bits com o segundo byte. | "Combino os dois bytes, formando o deslocamento completo." |
| **104** | `goto5` — `PC = OPC + H; fetch` | Calcula o endereço de destino FIM e busca seu opcode. | "Somo o deslocamento ao endereço do GOTO, apontando para FIM." |
| **105** | `goto6` — `goto Main1` | Retorna ao despacho, concluindo o desvio incondicional. | "O desvio está pronto; retorno a Main1 para executar FIM." |

**Checkpoint após a instrução:** PC aponta para o destino FIM (ILOAD c); o bloco MAIOR foi ignorado.

**Cuidado:** o `GOTO` é executado justamente para não cair no bloco `MAIOR`. O destino é `FIM`, onde se encontra `ILOAD c`.

### 4.21 Instrução IJVM 20 — `ILOAD c (resultado)` (clocks relativos 106–111)

**Objetivo:** Colocar c=3 na pilha para facilitar a verificacao.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **106** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **107** | `iload1` — `H = LV` | Copia a base das variáveis locais para H. | "Copio a base do quadro de variáveis, LV, para H." |
| **108** | `iload2` — `MAR = MBRU + H; rd` | Calcula o endereço de c e inicia leitura da memória. | "Calculo o endereço de c e solicito sua leitura na memória." |
| **109** | `iload3` — `MAR = SP = SP + 1` | Abre uma nova posição na pilha para o valor carregado. | "Incremento SP e MAR para reservar espaco na pilha." |
| **110** | `iload4` — `PC = PC + 1; fetch; wr` | Busca a próxima instrução e escreve na pilha o valor já lido. | "A leitura da variável já ocorreu; a escrita a coloca na nova posição da pilha." |
| **111** | `iload5` — `TOS = MDR; goto Main1` | Atualiza o topo para 3; retorna a Main1. | "TOS passa a valer 3. Com isso a instrução ILOAD termina." |

**Checkpoint após a instrução:** Pilha lógica: [3]; TOS=3 (0x00000003).

### 4.22 Instrução IJVM 21 — `HALT` (clocks relativos 112–113)

**Objetivo:** Encerrar a execução.

| Clock relativo | Microinstrução | O que conferir | Fala curta para a gravação |
|---:|---|---|---|
| **112** | `Main1` — `PC = PC + 1; fetch; goto (MBR)` | PC avança; inicia busca do byte seguinte; o MBR atual determina a próxima rotina. | "Este ciclo é o despacho: o PC avança, inicia uma busca e o MBR seleciona a rotina da instrução." |
| **113** | `halt1` — `goto halt1` | Microprograma entra na rotina de parada (loop de HALT). | "O HALT termina o programa e preserva o resultado 3 no topo da pilha." |

**Checkpoint após a instrução:** Programa encerrado; c=3 e TOS=3.

**Total da trilha a partir do primeiro `Main1`: 113 ciclos relativos até o primeiro `halt1`, contando `Main1` uma vez por instrução.** O ciclo `halt1` pode repetir enquanto a simulação estiver em estado de parada, portanto não continue contando suas repetições. A contagem absoluta do MIC1MMV pode conter ciclos de inicialização adicionais.

## 5. Interpretacao dos registradores ao longo da gravação

| Elemento | Como comentar ao aparecer |
|---|---|
| `PC` | Endereço **em bytes** do fluxo IJVM; `fetch` busca bytes de opcode/operando. |
| `MPC` | Endereço da microinstrução na memória de controle. Nas rotinas de opcode, os primeiros endereços fixados incluem `BIPUSH=0x10`, `ILOAD=0x15`, `ISTORE=0x36`, `IADD=0x60`, `ISUB=0x64`, `IFLT=0x9B`, `GOTO=0xA7`, `HALT=0xFF`; as etapas seguintes podem estar em outros endereços. |
| `MBR` / `MBRU` | Byte lido por `fetch`; `MBR` pode ser usado com sinal, `MBRU` sem sinal, por exemplo em índices locais. |
| `LV` / `H` | `LV` identifica a base das variáveis locais; `H` auxilia no calculo de `LV + índice`. |
| `MAR` / `MDR` | `MAR` indica a palavra acessada na memória; `MDR` guarda o valor lido ou a ser escrito; `rd` e `wr` controlam esses acessos. |
| `SP` / `TOS` | `SP` indica a posição da última palavra da pilha; `TOS` mantém o valor da palavra do topo. |
| `OPC` / `N` | `OPC` guarda temporariamente o valor testado pelo `IFLT`; a flag `N` indica se ele tem sinal negativo. Aqui `N=0`. |

### Endereços variáveis: como registrar com rigor

Se o professor exigir **endereços exatos** de `MPC`, `PC`, `MAR`, `SP` e `LV` em cada clique, não use números inferidos do modelo. Transcreva da sua tela, porque o posicionamento do programa na memória e o estado inicial determinam os endereços absolutos. Para cada bloco de aproximadamente 10 cliques, você pode preencher esta linha e anexar ao roteiro:

```text
Clock do simulador: ____ | Clock relativo do roteiro: ____
Microinstrução exibida: ________________________________
MPC=____  PC=____  MBR=____  MAR=____  MDR=____
SP=____   LV=____  H=____    TOS=____  OPC=____  N=____
Conclusão: _____________________________________________
```

**Como calibrar o roteiro com sua versão:** capture imagens ou um pequeno vídeo da interface no início e depois em blocos de 5–10 cliques. Com esses valores, é possível atualizar o Markdown com os endereços de cada clock real e alinhar a numeração absoluta, sem supor que o microcódigo carregado seja idêntico ao de referência.

## 6. Encerramento — fala pronta

> "A execução confirmou o resultado previsto. Inicialmente, com a igual a 2 e b igual a 5, a expressão duas vezes a menos b produziu c igual a menos 1. Para testar a condição, calculei 3 menos c, obtendo 4. Como esse número não é negativo, IFLT não desviou para MAIOR, e o bloco else somou 4, chegando a c igual a 3. A observação ciclo a ciclo mostrou como cada instrução IJVM é implementada por várias microinstruções que movimentam dados entre os registradores, usam a ULA e acessam a memória."

## 7. Checklist final

- [ ] Identificação do autor visível na tela durante toda a demonstração.
- [ ] Código `.jas` com dois `ILOAD a` aberto e mostrado.
- [ ] Montagem exibindo `assembly complete`, seguida de `Load`.
- [ ] Demonstração em modo IJVM e do resultado final `c=3`.
- [ ] Reinício em modo Clock, com registradores e microinstruções legíveis.
- [ ] Explicação dos ciclos de leitura (`rd`), escrita (`wr`), busca (`fetch`) e ULA.
- [ ] Registro visível de `c=-1` e do resultado da comparação `4`.
- [ ] Justificativa de `IFLT` não saltar para `MAIOR`.
- [ ] Desvio `GOTO FIM` e resultado final `TOS=3` destacados.
- [ ] Conclusão e encerramento da gravação.

## 8. Fontes técnicas de referência

- Microprograma `mic1ijvm.mal`: https://www.ontko.com/mic1/mic1ijvm.mal
- Especificação da linguagem IJVM: https://www.ontko.com/mic1/jas.html
- Introdução ao simulador MIC1MMV e modos de execução: https://csweb.wooster.edu/nsommer/cs210/mic1mmv-intro.html
- Especificação MAL: https://www.ontko.com/mic1/mal.html

