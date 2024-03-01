# Questionário Sistemas Embarcados I

## 1. Explique brevemente o que é compilação cruzada (***cross-compiling***) e para que ela serve.

Quando estamos trabalhando com sistemas embarcados, fazemos o código e compilamos o mesmo em um computador (host) de arquitetura x86, por exemplo, que é diferente do nosso microcontrolador que terá uma arquitetura ARM Cortex-M. Assim sendo, se passarmos unicamente o código redigido e compilado em computador para o microcontrolador, ele não irá conseguir compilar pela diferença de arquitetura e potência. Para isso, precisamos fazer o processo de compilação cruzada (cross-compiling), utilizando o GCC ARM Toolchain, que é o nosso pacote de software.

No curso de Sistemas Embarcados para poder compilar, utilizamos a chamada “arm-none-eabi-gcc”, que é o nosso compilador.

Especificando mais, ao compilar o arquivo main.c, rodamos a instrução:

`foo@bar$ arm-none-eabi-gcc -c -g -mcpu=cortex-m4 -mthumb -O0 -Wall main.c -o main.o`

- -c = se refere a compilar o arquivo de origem em um arquivo objeto
- -g =  solicitamos ao compilador para gerar informações de depuração/debug no arquivo objeto
- **-mcpu=cortex-m4 = especificação da arquitetura**
- **-mthumb = intrui a gerar o código na instrução thumb, conjunto de instruções de 16 bits**
- -O0 = desabilita otimizações (otimizações = 0)
- -Wall = ligar warnings a respeito de códigos que podem ser considerados questionáveis e que são de fácil detecção
- main.c = arquivo de origem
- -o = especifica o arquivo objeto
- main.o = é o nome do arquivo de saída, o arquivo de objeto compilado

## 2. O que é um código de inicialização ou ***startup*** e qual sua finalidade?

O código de inicialização, ou o startup.c como trabalho no curso é executado com a finalidade de preparar o ambiente de execução antes de inicializar a compilação do programa. Como não possuímos Sistema Operacional para coordenar a rotina intermediária antes de executar o main(), precisamos prover o código necessário, que é o startup.c.

Antes do main():

- Inicializar os *stacks*
    - Coloca o stack pointer para apontar para o topo da stack.
- Preparar os *streams* *stdin* / *stdout* / *stderr*;
    - stdin: fluxo para ler a entrada (geralmente associado ao teclado do computador). stdout: fluxo para enviar saídas (geralmente associado a tela do terminal). stderr: fluxo para mensagem de erro sejam tratadas de forma diferente.
- Adicionar ao stack os argumentos *argc* e *argv*, ou qualquer outro parâmetro passado através do *shell*, para que a função *main()* possa utilizá-los;
- Se necessário, copiar o conteúdo da seção *.data* (*initialized data*) da memória não volátil (flash) para a RAM;
    - .data : exemplo, int a = 10; vai para esse arquivo.
- Se necessário, copiar o conteúdo da seção *.fast* da memória não volátil (flash) para a RAM;
- Inicializa a seção *.bss* (*Block Started by Symbol*) em zero;
    - .bss armazena variáveis globais não inicializadas, por exemplo int b;, e o valor da variável é sempre incializado como 0 aqui. Essa seção então é inicializada co zero.
- Inicializa o *heap*;
    - área da memória para alocação dinâmica, variáveis no heap são acessíveis por qualquer parte do programa, e continuam até que sejam explicitamente desalocadas ou até a finalização do programa.
- Realiza qualquer função necessária para preparação do sistema operacional ou configuração que possa ser necessária;
- Chama a função *main()*.

No startup.c que realizamos, temos as seções:

- declaração e inicialização do ***Stack***;
    - é a pilha, memória temporária e pequena, e ela funciona com o stack pointer, que o movemos para alocação de memória, e a cada função chamada, suas variaveis são alocadas na stack e desalocadas segundo o princípio LIFO, last in first out.
    - utilização de PUSH, onde é decrementado o SP, e POP onde é incrementado.
- declaração e inicialização da Tabela de Vetores de Interrupção;
    - na arquitetura ARM Cortex-M, são reservados 15 vetores de interrupções. Sendo a posição 0 destinada ao Stack pointer.
- código do ***Reset Handler***;
    - copiar os dados da seção **.data** para a memória SRAM;
    - preencher com zero a seção **.bss**, e;
    - chamar a função **main()**.
- outros códigos ***Exception Handlers***.
    - NMI, SVC, PENDSV, SYSTICK…. heandlers.

## 3. Sobre o utilitário **make** e o arquivo **Makefile responda**:

### (a) Explique com suas palavras o que é e para que serve o **Makefile**.

O Makefile é um arquivo, que funciona como um script, que contém regras e definições que facilitam construir e compilar um projeto de software. Esse aquivo é usado pelo utilitário make que é uma forma de automatizar a compilação do software.

### (b) Descreva brevemente o processo realizado pelo utilitário **make** para compilar um programa.

O make começa lendo o arquivo Makefile, e começa a construir o primeiro target baseado nas regras. O make tqambém verifica as dependências entre os targets por exemplo, para determinar a melhor ordem de execução.

Uma regra no Makefile possui a construção:

targets: prerequisites
recipe

exemplo:

main.o: main.c
arm-none-eabi-gcc -c -g -mcpu=cortex-m4 -mthumb -O0 -Wall main.c -o main.o

O make irá compilar apenas os arquivos que estão desatualizados, para isso ele vê se o target está desatualizado (ou seja, se el não existe, ou se é mais antigo que qualquer prerequisite). 

Assim definido o que é necessário compilar, ele começa a execução das recipes dos targets.

### (c) Qual é a sintaxe utilizada para criar um novo **target**?

targets: prerequisites
recipe

exemplo:

main.o: main.c
arm-none-eabi-gcc -c -g -mcpu=cortex-m4 -mthumb -O0 -Wall main.c -o main.o

### (d) Como são definidas as dependências de um **target**, para que elas são utilizadas?

As dependências são escritas no Makefile, dentro da regra do target.

No exemplo 

main.o: main.c
arm-none-eabi-gcc -c -g -mcpu=cortex-m4 -mthumb -O0 -Wall main.c -o main.o

O target main.o depende do main.c, ou seja, precisa ser reconstruído se main.c for modificado e é gerado através dele.

### (e) O que são as regras do **Makefile**, qual a diferença entre regras implícitas e explícitas?

Uma regra informa o make duas coisas: quando um target está desatualizado, através da comparação com o prerequisite determinado, e como fazer para atualizá-lo.

targets: prerequisites
recipe

Essas regras, definidas pelo usuário no Makefile são as regras explícitas. Já as regras implícitas estão relacionadas a quando e como refazer uma classe de arquivos por meio dos seus nomes. São regras inferidas pelo make com base em convenções padrões, por exemplo que o make reconhece a extensão .c e compila o main.c usando o compilador gcc.

## 4. Sobre a arquitetura **ARM Cortex-M** responda:

### (a) Explique o conjunto de instruções ***Thumb*** e suas principais vantagens na arquitetura ARM. Como o conjunto de instruções ***Thumb*** opera em conjunto com o conjunto de instruções ARM?

O conjunto de instruções do ARM tradicional tem 32 bits de enderaçamento (endereços com 32 bits), enquanto o conjunto de instruções Thumb é reduzido e possui 16 bits. O Thumb-2 já possui 32 bits de endereçamento. Ou seja, o Thumb complementa o conjunto de instruções ARM e oferece uma versão mais compactada de instruções. 

Por ser assim, ofere um código mais compactado, economia de memória e eficiência de energia. No ARM Cortex-M o conjunto de instruções Thumb é frequentemente a base em si do conjunto de instruções, e também o Thumb-2, que é parecido com o conjunto do ARM tradicional, mas ainda assim mais eficiente. O Thumb-2, com instruções de 32 bits, aumenta a capacidade de processamento quando necessário.

### (b) Explique as diferenças entre as arquiteturas ***ARM Load/Store*** e ***Register/Register***.

Load/Store está mais relacionado com arquiteturas RISC, onde as operações ULA com os dados são realizadas principalmente com registradores. Load é sobre move dados da memória para registrador, e Store de mover dados de um registrador para memória. 

Já o Register/Memória* é mais relacionado a arquitetura CISC, as instruções atuam diretamente em dados da memória, sem precisar primeiro passar para um registrador e então realizar a operação como no Load/Store. A operação pode acontecer também diretamente na memória.

### (c) Os processadores **ARM Cortex-M** oferecem diversos recursos que podem ser explorados por sistemas baseados em **RTOS** (***Real Time Operating Systems***). Por exemplo, a separação da execução do código em níveis de acesso e diferentes modos de operação. Explique detalhadamente como funciona os níveis de acesso de execução de código e os modos de operação nos processadores **ARM Cortex-M**.

Os níveis de acesso de execução do código podem ser baseados no Kernel x User mode de RTOS, onde no primeiro, que é o de acesso privilegiado, tenho acesso a registradores e outros recursos, utilizamos o Main Stack Pointer. Já no mde de usuário, tenho restrições de acesso aos mesmos., e usamos o Program Stack Pointer.

Os modos de operação são o Thread e Hadler Mode. O primeiro geralmente é quando executando o código, já o segundo geralmente é acionado quando há uma exceção para atenda a ISR. No Thread Mode posso ter ou não acesso privilegiado, e geralmente o processador incia nesse modo com nível de privilégio total.

### (d) Explique como os processadores ARM tratam as exceções e as interrupções. Quais são os diferentes tipos de exceção e como elas são priorizadas? Descreva a estratégia de **group priority** e **sub-priority** presente nesse processo.

Reset: rotina de iniciar o sistema

NMI (Non-Maskable Interrupt) : é uma exceção de alta prioridade

HardFault: exceção decorrente de uma condição de erro na execução de uma instrução, por exemplo acesso a memória inválido, erro de divisão por zero

SVCall: exceção causada pela chamada da instrução do Supervisor Call

PendSV & SysTickt: exceção desencadeada pelo software

Exceções Externas geralmente relacionada a periféricos.

A prioridade de exceções externas é gerenciadas pelo componente NVIC, e podem estar em categorias de sub priority ou priority, que são configuradas pelo integrador.

### (e) Qual a diferença entre os registradores **CPSR** (***Current Program Status Register***) e **SPSR** (***Saved Program Status Register***)?

O CPSR é o registrador sobre o estado atual do processador. Já o SPSR é usado para salvar os dados do CPSR quando ocorre uma exceção.

### (f) Qual a finalidade do **LR** (***Link Register***)?

O LR é o Link Register e é usado principalmente quando há uma interrupção, onde parte do contexto atual é salvo nele. Ao retornar de uma interrupção, ele é decodificado e informa sobre qual a atualização realizar no Program Counter (endereço de retorn) para voltar aonde estava, qual modo se deve utilizar (MSP, PSP…).

### (g) Qual o propósito do Program Status Register (PSR) nos processadores ARM?

O PSR é dividie em 3 registros: Application PSR, Execution PSR, Interrupt PSR. Ele tem informações sobre o estado atual do processador.

APSR: Negative flag, Zero flag, Carry out, Overflow.

IPSR: número da exceção que está sendo tratada. Se está em estado normal, esse valor é 0.

EPSR: Thumb state.

### (h) O que é a tabela de vetores de interrupção?

A Tabela de Vetores de Interrupção lista identificadores a endereços de memória ligados a determinadas funções. O processador recorre a essa tabela para identificar e acionar a função apropriada, de acordo com o evento que ocorrer. Muito usado para gestão de interupções

### (i) Qual a finalidade do NVIC (**Nested Vectored Interrupt Controller**) nos microcontroladores ARM e como ele pode ser utilizado em aplicações de tempo real?

O NVIC é um componente usado para gerenciar interrupções internas ou externas, e auxiliar na eficiência da CPU para tratá-las. Ele prioriza as interrupções, e assim auxilia também a reduzir a latência de interrupção. O NVIC garante que interrupções de maior prioridade sejam executadas totalmente antes das de menor priorirdade.

### (j) Em modo de execução normal, o Cortex-M pode fazer uma chamada de função usando a instrução **BL**, que muda o **PC** para o endereço de destino e salva o ponto de execução atual no registador **LR**. Ao final da função, é possível recuperar esse contexto usando uma instrução **BX LR**, por exemplo, que atualiza o **PC** para o ponto anterior. No entanto, quando acontece uma interrupção, o **LR** é preenchido com um valor completamente diferente, chamado de **EXC_RETURN**. Explique o funcionamento desse mecanismo e especifique como o **Cortex-M** consegue fazer o retorno da interrupção.

Durante a execução normal, o Cortex M usa a instrução BL para chamar funções, alterando o Program Counter para o endereço da função e salva o endereço de retorno no Link Register. Para retornar, usa-se BX LR, que restaura o PC ao estado prévio. No entanto, em uma interrupção, o LR adquire um valor especifico, EXC_RETURN, indicando o contexto de execução antes da interrupção e permitindo que o processador retome corretamente ao ponto interrompido depois de tratat a interrupção..

### (k) Qual a diferença no salvamento de contexto, durante a chegada de uma interrupção, entre os processadores Cortex-M3 e Cortex M4F (com ponto flutuante)? Descreva em termos de tempo e também de uso da pilha. Explique também o que é ***lazy stack*** e como ele é configurado.

A diferença está no tratamento dos registros de ponto flutuante pelo M4F, que possui uma unidade de ponto flutuante (FPU). O Cortex M4F usa a técnica de "lazy stacking" para evitar ao máximo o salvamento dos registros de ponto flutuante na stack, a menos que seja necessário. Se uma função que utiliza ponto flutuante não é chamada durante o evento/interrupção, os registradores de ponto flutuante não serão salvos, economizando tempo e espaço na stack. Assim, reduz o tempo de interrupção e o uso da stack comparado ao processo padrão.
