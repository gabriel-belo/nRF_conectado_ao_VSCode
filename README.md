# nRF_conectado_ao_VSCode
Aprendendo sobre como utilizar o nRF no VS Code

### Função de cada Parte no nRF:

* nRF Connect SDK: Contém o Zephyr RTOS e as bibliotecas da Nordic. É a parte principal do seu ambiente de desenvolvimento. Se algum arquivo estiver corrompido, a reinstalação resolve.

* nRF Connect SDK Toolchain: É o compilador (GCC) e o conjunto de ferramentas que transformam seu código em um arquivo executável. Uma instalação corrompida na toolchain é uma causa comum de erros de compilação.

* nRF Command Line Tools: Contém apenas ferramentas de linha de comando para interagir com a placa (fazer o flash, apagar, etc.). Como você não está usando uma placa física, esse pacote não está envolvido na falha de compilação para native_sim ou QEMU.


## Documentos para uma Estrutura de Projeto Profissional
Para construir uma estrutura de projeto profissional no Zephyr, você precisa entender como os arquivos Kconfig, devicetree e CMakeLists.txt funcionam em conjunto.

### Guia do Zephyr para Kconfig e Device Tree:

* Kconfig - Configuração de Sistema: https://docs.zephyrproject.org/latest/build/kconfig/index.html

* O que é: O Kconfig é um sistema de configuração para definir quais funcionalidades, drivers e recursos do Zephyr serão incluídos na sua compilação.

* O que deve conter: Seu arquivo prj.conf (geralmente localizado na raiz do projeto) conterá linhas como CONFIG_I2C=y para habilitar a interface I2C ou CONFIG_BT=y para habilitar o Bluetooth.

### Device Tree - Descrição de Hardware: https://docs.zephyrproject.org/latest/build/dts/index.html

* O que é: O Device Tree (DT) é a forma padrão no Zephyr para descrever seu hardware. Ele mapeia os pinos da MCU para os seus componentes.

* O que deve conter: Seu arquivo your_board.dts ou your_board.overlay irá descrever como seus sensores (adxl362, bmi270) estão conectados ao nRF9160, especificando a interface (I2C) e o endereço.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------## Variáveis de ambiente
Em projetos com Zephyr, as variáveis de ambiente são valores que você define fora do seu código-fonte, mas que são usados pelo sistema de compilação (CMake e o Zephyr Build System) para configurar o seu projeto. Elas funcionam como chaves globais que podem ser acessadas por diferentes partes do processo de build.

Elas são particularmente úteis para:
* Definir o hardware de destino: Informar para qual placa o projeto deve ser compilado.
* Configurar o caminho para ferramentas: Apontar onde o Zephyr SDK ou o compilador estão instalados.
* Passar informações de configuração: Adicionar flags de compilação, ou desabilitar funcionalidades.

#### Como as Variáveis de Ambiente se Encaixam no Zephyr
No Zephyr, as variáveis de ambiente mais importantes são usadas para indicar ao sistema de compilação qual board (placa) e qual toolchain (conjunto de ferramentas de compilação) usar.

Aqui estão as mais comuns:

ZEPHYR_BASE

Esta variável aponta para o diretório raiz da sua instalação do Zephyr. O sistema de build usa esse caminho para encontrar todos os arquivos do kernel, drivers e bibliotecas do Zephyr que seu projeto precisa.

Exemplo de uso: Você pode definir ZEPHYR_BASE no seu terminal, como:

Bash

export ZEPHYR_BASE=/caminho/para/seu/zephyrproject/zephyr
(No Linux/macOS) ou em um script setup.bat no Windows.

ZEPHYR_TOOLCHAIN_VARIANT

Esta variável especifica qual tipo de toolchain (conjunto de compiladores, debuggers, etc.) você está usando. Por exemplo, você pode usar gnuarmemb para o compilador ARM ou o zephyr para o SDK oficial do Zephyr.

Exemplo de uso:

Bash

export ZEPHYR_TOOLCHAIN_VARIANT=zephyr
BOARD

Esta é uma das variáveis mais importantes. Ela informa ao sistema de build para qual placa de desenvolvimento o código deve ser compilado. O valor da variável corresponde ao nome de um diretório dentro de zephyr/boards/.

Exemplo de uso: Para o seu projeto com o nRF9160, você usaria o nome da sua placa (por exemplo, nrf9160dk_nrf9160).

Bash

export BOARD=nrf9160dk_nrf9160

Como Usar na Prática

Normalmente, você não precisa definir essas variáveis manualmente toda vez. A melhor prática é usar o Zephyr West (o CLI, ou command-line tool) para gerenciar isso.

Quando você usa um comando como:

Bash

west build -b nrf9160dk_nrf9160

O west faz todo o trabalho para você. Ele:

* Verifica o nome da board (nrf9160dk_nrf9160).

* Procura a configuração daquela placa.

* Define automaticamente a variável de ambiente BOARD para nrf9160dk_nrf9160.

* Executa o CMake e o compilador com as configurações corretas para essa placa.

### Formas de definir variáveis de ambiente
Pense nas variáveis de ambiente como atalhos globais. Em vez de digitar um caminho longo toda vez, você pode usar uma variável curta. A documentação mostra três maneiras de fazer isso, cada uma para um caso de uso diferente.

#### Opção 1: Apenas para a Sessão Atual do Terminal

* Essa opção é a mais simples e a mais temporária.

* Como funciona: Você digita o comando set MY_VARIABLE=foo (no Windows) ou export MY_VARIABLE=foo (no Linux/macOS).

* Quando usar: É perfeita para testes e experimentação rápida. A variável existe apenas enquanto a janela do terminal estiver aberta. Assim que você a fechar, a variável desaparece.

* O que a documentação diz: Aviso: Isso é melhor para experimentação. Isso significa que não é uma solução permanente para o seu dia a dia.

#### Opção 2: Para Todos os Terminais (Configuração Permanente)

* Essa é a solução ideal se você quer que uma variável esteja sempre disponível, não importa quantas janelas de terminal você abra.

* Como funciona (Windows): Você usa o comando setx (setx MY_VARIABLE foo). Isso salva a variável no sistema operacional. Para que a alteração tenha efeito, você precisa fechar e reabrir o terminal.

* Como funciona (Linux/macOS): Você adiciona a linha de exportação (export MY_VARIABLE=foo) a um arquivo de inicialização do seu shell (como .bashrc ou .zshrc). Isso faz com que a variável seja carregada automaticamente toda vez que você abrir um novo terminal.

* Quando usar: Quando você tem uma configuração que sempre usa, como o caminho para o SDK do Zephyr. Por exemplo, o setx é a forma de definir o caminho do ZEPHYR_SDK_INSTALL_DIR permanentemente.

#### Opção 3: Usando Arquivos .zephyrrc (A Solução do Zephyr)

* Essa é a maneira mais recomendada para projetos Zephyr, pois ela é específica e não afeta outras partes do seu sistema.

* Como funciona: Você cria um arquivo chamado zephyrrc.cmd (no Windows) ou zephyrrc.sh (no Linux/macOS) no seu perfil de usuário. Dentro desse arquivo, você define suas variáveis, por exemplo: set MY_VARIABLE=foo.

* O que é o pulo do gato: A variável só é carregada quando você executa um script específico do Zephyr, o zephyr-env.cmd (ou zephyr-env.sh).

* Quando usar: É a melhor opção se você não quer que suas variáveis Zephyr "poluam" seu ambiente global, ou se você trabalha com diferentes versões do Zephyr em que a variável ZEPHYR_BASE precisa mudar dependendo do projeto.

* Vantagem: Você tem um ambiente de desenvolvimento Zephyr totalmente isolado e configurado, sem a necessidade de definir variáveis globais para todo o sistema operacional.

#### Scripts de Ambiente Zephyr (zephyr-env)

O texto também menciona os scripts zephyr-env.sh (Linux/macOS) e zephyr-env.cmd (Windows). Esses scripts são a forma mais prática de configurar seu ambiente. Eles fazem três coisas importantes:

* Definem o ZEPHYR_BASE: Apontam para o local exato do seu repositório Zephyr.

* Modificam o PATH: Adicionam as pastas de ferramentas do Zephyr ao seu caminho de busca, permitindo que você execute comandos do Zephyr de qualquer lugar no terminal.

* Carregam o zephyrrc: Se você seguiu a Opção 3, é esse script que lê o arquivo .zephyrrc e carrega suas variáveis personalizadas.

Em resumo, as variáveis de ambiente são uma ferramenta para o processo de build do Zephyr. O texto mostra a você três formas de gerenciá-las, da mais temporária à mais permanente e específica, sendo a Opção 3 (com os scripts e o arquivo .zephyrrc) a abordagem mais limpa e profissional para o desenvolvimento com Zephyr.

#### Resumo
As variáveis de ambiente no Zephyr são uma camada de abstração que separa as configurações do seu ambiente de desenvolvimento do seu código-fonte. Em vez de "hard-codar" o nome da sua placa no seu programa, você simplesmente a define como uma variável, e o sistema de build se encarrega de usar a configuração correta. Isso torna seu projeto mais flexível e portátil, pois a mesma base de código pode ser compilada para diferentes placas, simplesmente alterando o valor de uma variável.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Desenvolvimento de aplicativos

O desenvolvimento de aplicativos com o Zephyr é uma abordagem diferente do que a maioria das pessoas está acostumada em ambientes como Android ou iOS. Em vez de criar um aplicativo que roda sobre um sistema operacional de propósito geral, no Zephyr você está construindo um aplicativo de firmware que roda diretamente sobre o hardware, com o auxílio do kernel Zephyr.


### Guia do Zephyr para o Sistema de Build (CMake):

* Visão Geral do CMake: https://docs.zephyrproject.org/latest/build/cmake/index.html

* O que é: O CMakeLists.txt é o arquivo que gerencia o processo de compilação.

* O que deve conter: Ele define qual código-fonte (.c) e quais dependências (bibliotecas do Zephyr) seu projeto precisa. O Zephyr tem um arquivo CMakeLists.txt de modelo, e você só precisa modificá-lo para adicionar seus arquivos-fonte e as dependências de sua aplicação.

## Criação de uma aplicação:
1. Criar uma configuração de construção:
Informa ao compilador quais arquivos da placa incluir na construção, tornando a saída compatível com a placa.
3. 
