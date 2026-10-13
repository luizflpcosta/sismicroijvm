# Mini projeto - Sistemas Microprocessados

## Objetivo da gravação
Demonstrar a tradução de um programa de alto nível para IJVM e sua execução no simulador MIC1MMV, incluindo a inspeção das instruções, das microinstruções e dos registradores em nível de clock.

**Identificação:** exibir nome completo, disciplina e turma em um texto sobreposto na tela durante toda a execução. Não é necessário mostrar CPF completo.

## 1. Apresentação

**Fala sugerida:**
> Neste mini projeto de Sistemas Microprocessados, vou demonstrar a tradução de um programa de alto nível para a linguagem IJVM e sua execução no simulador MIC1MMV. O programa utiliza o último dígito do meu CPF, que é 2, e o último dígito da minha matrícula, que é 5. A partir desses valores, calculamos c igual a duas vezes a menos b e executamos um desvio condicional.

Mostrar o código original e explicar o resultado previsto: a=2, b=5, c=2*2-5=-1. Como -1 não é maior que 3, ocorre o `else` e o resultado final esperado é 3.

## 2. Explicação da tradução para IJVM

**Fala sugerida:**
> O conjunto básico da IJVM é baseado em pilha. Para armazenar valores em variáveis locais, utilizo BIPUSH e ISTORE. Para recuperá-los, utilizo ILOAD. Como não existe IMUL no conjunto básico utilizado aqui, a expressão duas vezes a foi implementada com ILOAD a, DUP e IADD. Depois carrego b, faço a subtração com ISUB e armazeno o resultado em c.

Destacar no editor as instruções de inicialização, duplicação, soma e subtração.

**Fala sugerida para o condicional:**
> Para testar se c é maior que 3, eu calculo 3 menos c. Se o resultado for negativo, significa que c é maior que 3; nesse caso, IFLT realiza o salto para MAIOR. Caso contrário, a execução continua pelo bloco else.

## 3. Carregamento no simulador

1. Abrir `Mic1MMV_hr.jar` ou `Mic1MMV_lr.jar` usando Java.
2. Escolher `File > Assemble / Load JAS File...` e selecionar `mini_projeto_mic1_ijvm.jas`.
3. Conferir `assembly complete` e clicar em `Load`.
4. Selecionar `IJVM` no seletor de velocidade e avançar instrução por instrução com a seta de execução.

**Fala sugerida:**
> O programa foi montado pelo assembler sem erros e carregado no simulador. Primeiro vou acompanhar as instruções da IJVM para verificar a lógica do algoritmo. Em seguida, vou reiniciar a execução e observar as microinstruções com a opção Clock.

## 4. Pontos de verificação em modo IJVM

| Após a instrução | Verificação esperada |
|---|---|
| `ISTORE a` | Variável `a` recebe 2. |
| `ISTORE b` | Variável `b` recebe 5. |
| `ILOAD a` / `DUP` / `IADD` | O valor 2 é duplicado na pilha e somado, produzindo 4. |
| `ILOAD b` / `ISUB` | É calculado 4 - 5 = -1; representação em 32 bits: `0xFFFFFFFF`. |
| `ISTORE c` (primeiro) | A variável `c` recebe -1. |
| `BIPUSH 3` / `ILOAD c` / `ISUB` | É calculado 3 - (-1) = 4. |
| `IFLT MAIOR` | Não realiza o salto: 4 não é negativo. |
| `ILOAD c` / `BIPUSH 4` / `IADD` | É calculado -1 + 4 = 3. |
| `ISTORE c` (segundo) | `c` passa a valer 3. |
| `GOTO FIM` | Desvia para o final, sem executar o bloco `MAIOR`. |
| `ILOAD c` / `HALT` | Topo da pilha mostra 3 (`0x00000003`); programa encerra. |

**Observação:** o `OUT` convencional imprime caracteres ASCII, não números decimais diretamente. A versão principal demonstra o valor 3 no topo da pilha, sem utilizar saída textual.

## 5. Execução em nível de clock

1. Reiniciar a simulação com o comando de reset do programa.
2. Selecionar `Clock` na caixa de velocidade, em vez de `IJVM`.
3. Avançar lentamente e observar a **microinstrução corrente**, o **MPC** e as mudanças dos registradores.
4. Analisar especialmente `BIPUSH`, `ILOAD`, `IADD`, `ISUB`, `ISTORE` e `IFLT`.
5. Mostrar como uma instrução IJVM requer vários ciclos de clock; opcionalmente alternar para `SubClock` para visualizar as etapas internas de um ciclo.

**Fala sugerida:**
> Agora estou no nível de clock. Cada acionamento avança a execução do microprograma responsável por implementar a instrução IJVM. O MPC indica a posição no microprograma; o PC acompanha os bytes do programa; o MBR recebe os bytes buscados; SP identifica o topo da pilha e TOS mantém o valor do topo. MAR e MDR participam das operações de acesso à memória, e H é utilizado como registrador auxiliar em operações da ULA. Vou observar as microinstruções que modificam esses registradores para acompanhar os valores do cálculo.

**Exemplo de fala ao chegar ao IADD:**
> Neste ponto, a instrução IADD soma dois operandos da pilha. Esses operandos são duas cópias do valor 2, carregadas por ILOAD a e DUP. A operação produz 4 e atualiza o topo da pilha. Essa operação IJVM não ocorre em um único ciclo; são necessárias várias microinstruções.

**Exemplo de fala ao chegar ao desvio:**
> Aqui foi calculado 3 menos c, ou seja, 4. Como esse valor não é negativo, a condição do IFLT não é satisfeita e a execução prossegue pelo bloco else. Depois do IADD final, c recebe o valor 3.

> Atenção: os valores exatos de MPC, PC, SP e MAR variam conforme o carregamento e o microprograma utilizado. Relate os números exibidos no seu simulador, em vez de antecipar endereços.

## 6. Conclusão

**Fala sugerida:**
> A simulação confirmou a lógica do programa original: com a igual a 2 e b igual a 5, obtive inicialmente c igual a menos 1. Como essa quantidade não é maior que 3, o programa executou o else e somou 4, chegando ao resultado final 3. A execução em nível de clock permitiu observar que cada instrução da IJVM é realizada por uma sequência de microinstruções, que controlam registradores, ULA, pilha e memória na MIC-1.

## Checklist da gravação

- [ ] Identificação do autor permanece visível durante toda a execução.
- [ ] Código fonte `.jas` é mostrado.
- [ ] Compilação apresenta `assembly complete`.
- [ ] O programa é carregado e executado em modo `IJVM`.
- [ ] O resultado intermediário `c=-1` é demonstrado.
- [ ] O `IFLT` não realiza o salto, justificando o uso do `else`.
- [ ] O resultado final `c=3` aparece no topo da pilha.
- [ ] O modo `Clock` mostra a sequência de microinstruções e registradores.
- [ ] A gravação é encerrada com uma conclusão.
