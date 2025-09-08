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

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Variáveis de ambiente
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

Um "aplicativo" no Zephyr é o firmware que você escreve, compilado junto com o kernel Zephyr, para rodar em um dispositivo embarcado. Diferente de um aplicativo de smartphone, que é instalado em um sistema operacional que já está lá (como o Android), o aplicativo Zephyr se torna o próprio sistema operacional.

Pense nisso como um único pacote de software feito sob medida para a sua placa: ele inclui o núcleo do Zephyr (que gerencia as tarefas e os recursos) e o seu código (a lógica da sua aplicação). Esse pacote é tudo o que a sua placa precisa para funcionar.

O desenvolvimento de aplicativos com o Zephyr é uma abordagem diferente do que a maioria das pessoas está acostumada em ambientes como Android ou iOS. Em vez de criar um aplicativo que roda sobre um sistema operacional de propósito geral, no Zephyr você está construindo um aplicativo de firmware que roda diretamente sobre o hardware, com o auxílio do kernel Zephyr.

### O Ciclo de Desenvolvimento de um Aplicativo Zephyr
O desenvolvimento de aplicativos em Zephyr segue um fluxo bem definido, impulsionado por ferramentas e conceitos específicos.

### 1. Configuração do Hardware (Device Tree)
Antes de escrever qualquer código, você descreve seu hardware em um arquivo de texto chamado Device Tree. Este arquivo atua como um mapa, dizendo ao Zephyr exatamente como seu circuito está montado. Ele especifica:

Periféricos: Seus sensores, LEDs, botões e outros componentes.

Conexões: Quais pinos do microcontrolador estão ligados a quais componentes.

Interfaces: Se você usa I2C, SPI, UART, etc., e em qual barramento cada dispositivo está.

Essa abordagem separa a descrição do hardware do seu código, tornando-o mais portável. Se você mudar um sensor de pino, basta alterar o Device Tree, não seu código C.

### 2. Definição das Funcionalidades (Kconfig)
Em seguida, você usa o Kconfig para selecionar quais funcionalidades de software você precisa. O Zephyr é modular, então você só inclui o que vai usar. A configuração é feita em um arquivo chamado prj.conf com linhas simples:

CONFIG_BT=y: Habilita o subsistema Bluetooth.

CONFIG_I2C=y: Habilita o driver I2C para comunicação.

O sistema de compilação usa essas configurações para incluir apenas o código necessário do Zephyr na sua aplicação, o que economiza memória e espaço de armazenamento, recursos escassos em dispositivos embarcados.

### 3. Escrevendo a Lógica da Aplicação
A maior parte do seu trabalho se concentra em escrever a lógica do seu aplicativo em arquivos C, começando pelo main.c. Aqui, você usará as APIs do Zephyr.

O que são APIs? No Zephyr, as APIs são um conjunto de funções C prontas para uso que permitem interagir com o hardware. Em vez de escrever o código para controlar um pino digital, você simplesmente chama a função gpio_pin_set(). As APIs abstraem a complexidade e fornecem uma interface consistente para todos os drivers e funcionalidades do Zephyr.

Organização do Código: Para projetos maiores, você pode criar múltiplos arquivos .c para separar a lógica (por exemplo, um arquivo para lidar com sensores, outro para comunicação, etc.). O sistema de build CMake é usado para gerenciar todos esses arquivos e suas dependências.

### 4. Multitarefas (Threads)
O Zephyr é um sistema operacional multitarefa, o que significa que ele pode executar várias tarefas de forma quase simultânea. Você pode organizar sua aplicação em threads, que são sequências de código que o kernel executa de forma independente. Por exemplo:

* Uma thread pode ser responsável por ler dados do sensor a cada 5 segundos.

* Outra thread pode gerenciar a conexão com a rede celular.

* Uma terceira thread pode ficar em estado de espera, acordando apenas quando um botão é pressionado.

Usar threads ajuda a organizar o código, melhora a capacidade de resposta do sistema e permite um gerenciamento mais eficiente do consumo de energia.

### 5. Compilação e Gravação

Finalmente, você usa a ferramenta de linha de comando west para compilar todo o projeto. O west lê as configurações do Kconfig e o mapa do Device Tree para gerar um único arquivo de firmware. Depois, você usa o mesmo west para gravar esse firmware na sua placa, substituindo qualquer software que estivesse lá antes.

### Guia do Zephyr para o Sistema de Build (CMake):

* Visão Geral do CMake: https://docs.zephyrproject.org/latest/build/cmake/index.html

* O que é: O CMakeLists.txt é o arquivo que gerencia o processo de compilação.

* O que deve conter: Ele define qual código-fonte (.c) e quais dependências (bibliotecas do Zephyr) seu projeto precisa. O Zephyr tem um arquivo CMakeLists.txt de modelo, e você só precisa modificá-lo para adicionar seus arquivos-fonte e as dependências de sua aplicação.


### arquivos em um aplicativo Zephyr simples:

<app>
├── CMakeLists.txt
├── app.overlay
├── prj.conf
├── VERSION
└── src
    └── main.c

Esses conteúdos são:

* ***CMakeLists.txt:*** Ele não é um código que roda no seu dispositivo, mas sim um conjunto de instruções que informa ao CMake (o sistema de compilação) como transformar seus arquivos de código em um firmware executável. Pense no CMakeLists.txt como a lista de ingredientes e o manual de instruções para a sua receita.

#### Qual a funcionalidade do CMakeLists.txt?
1. Encontrar Seus Arquivos: A função mais básica do CMakeLists.txt é dizer ao compilador onde estão os seus arquivos de código-fonte (.c, .h). Ele lista todos os arquivos que precisam ser compilados e ligados para formar o firmware final.

2.  Ligar ao Sistema de Compilação do Zephyr: Este é o ponto mais crucial. O Zephyr tem um sistema de compilação complexo e bem otimizado. Ao "vincular" seu projeto a ele, o CMakeLists.txt ganha acesso a todas as funcionalidades internas do Zephyr, como:

* Configurações do Kconfig: O CMake lê o seu arquivo prj.conf para saber quais drivers e funcionalidades devem ser incluídos na compilação.

* Mapas do Device Tree: Ele processa os arquivos .dts e .overlay para entender a sua configuração de hardware e incluir os drivers corretos.

* Ferramentas de Flash/Debug: Ele configura as ferramentas para que você possa usar comandos simples como west flash ou west debug, que o west usa para gravar o firmware ou iniciar uma sessão de depuração.

#### Um exemplo prático
Imagine que você tem um projeto com a seguinte estrutura de arquivos:

my_app/
├── CMakeLists.txt
├── prj.conf
├── main.c
├── sensor_reader.c
└── sensor_reader.h
Seu CMakeLists.txt teria uma linha parecida com esta:

CMake

´´´
// Adiciona os arquivos .c do seu aplicativo
target_sources(app PRIVATE main.c sensor_reader.c)
´´´

Essa linha é uma instrução para o CMake. Ela diz: "O seu projeto (app) tem dois arquivos de código-fonte: main.c e sensor_reader.c. Inclua-os na compilação."

Quando você executa o comando west build, o West passa essa instrução para o CMake, que então usa todas as informações do Zephyr para compilar seu código e criar o arquivo de firmware final. Sem o CMakeLists.txt, o sistema de compilação não saberia o que compilar.


* ***app.overlay***:

#### Qual a funcionalidade do app.overlay?

* Personalização de Hardware: Este arquivo é onde você define como os componentes externos do seu circuito estão conectados ao microcontrolador. Por exemplo, se você adicionou um sensor de temperatura, um LED extra, ou um botão que não faz parte da placa original, você descreve essas conexões no app.overlay.

* Sobrescrever o Devicetree Base: O Zephyr já tem uma descrição (.dts) para cada placa de desenvolvimento que ele suporta (como a nrf9160dk_nrf9160). Este arquivo base descreve os componentes que já vêm na placa. O app.overlay permite que você modifique ou adicione informações a essa descrição.

* Habilitar Periféricos: É comum usar o app.overlay para habilitar ou desabilitar periféricos de hardware que estão no chip, mas que o devicetree base pode ter desativado. Por exemplo, você pode habilitar o barramento I2C para que seu sensor possa ser detectado pelo sistema.

#### Exemplo Prático
Imagine que a descrição padrão da sua placa nrf9160dk_nrf9160 já define os LEDs e botões que vêm de fábrica. Agora, você adicionou um sensor de movimento ADXL362 e conectou-o ao barramento I2C.

Seu arquivo app.overlay teria um conteúdo parecido com este:

/* Ativa o barramento I2C, que pode estar desabilitado no arquivo base */
´´´
&i2c1 {
    status = "okay";

    /* Descreve o sensor conectado ao I2C */
    adxl362@53 {
        compatible = "adi,adxl362";
        reg = <0x53>;
        label = "ADXL362";
    };
};
´´´

Neste exemplo:

* &i2c1: Indica que estamos fazendo alterações no nó (seção) chamado i2c1 no devicetree base.

* status = "okay": Garante que o barramento I2C esteja ativado e pronto para ser usado.

* adxl362@53: Adiciona um novo dispositivo (adxl362) ao barramento I2C, especificando que ele é compatível com o driver adi,adxl362 e que seu endereço na rede I2C é 0x53.

Quando você compila seu projeto, o sistema de build do Zephyr lê primeiro o devicetree original da sua placa e, em seguida, aplica as alterações do seu app.overlay. O resultado é um único arquivo de devicetree completo que descreve toda a sua configuração de hardware. Isso é o que permite que o sistema de compilação gere o firmware correto para o seu circuito.


* ***prj.conf***: Este arquivo é para configuração, não para código-fonte. Você vai colocar aqui as linhas que habilitam funcionalidades do Zephyr. Por exemplo, CONFIG_SENSOR=y para usar a API de sensores, ou CONFIG_BT=y para Bluetooth.

#### Qual a funcionalidade do prj.conf?
* A principal função do prj.conf é permitir que você configure as funcionalidades do software de forma modular e eficiente. Em vez de escrever código para cada funcionalidade, você simplesmente as habilita no prj.conf, e o sistema de compilação faz o resto.

* Habilitar Módulos e Drivers: Você usa o prj.conf para ligar e desligar módulos inteiros do Zephyr. Por exemplo, se você precisa usar o Bluetooth, adicione a linha CONFIG_BT=y. Se você não precisa, não a adicione, e o Zephyr não incluirá o código do Bluetooth, economizando memória.

* Configurar Parâmetros de Módulos: Além de habilitar, você pode ajustar parâmetros específicos de cada módulo. Por exemplo, se você habilitou o módulo de gerenciamento de energia, pode definir a linha CONFIG_PM_DEVICE=y para ativá-lo, e depois ajustar o tempo de sono do sistema com CONFIG_PM_DEVICE_SUSPEND_DELAY=1000.

* Mesclar com Outras Configurações: O texto menciona que o prj.conf é um "fragmento Kconfig". Isso significa que ele não é a única fonte de configurações. O sistema de compilação do Zephyr pega as configurações de vários lugares (como as configurações padrão da sua placa e as configurações do próprio kernel) e mescla tudo com as suas configurações no prj.conf. As suas configurações têm prioridade, garantindo que o resultado final seja exatamente o que você precisa para o seu projeto.

#### Exemplo Prático
Imagine que você está construindo um projeto com o nRF9160 que usa um sensor I2C e precisa se conectar à rede celular. Seu arquivo prj.conf poderia ter o seguinte conteúdo:

/* Habilita o suporte a I2C para comunicação com o sensor */
CONFIG_I2C=y

/* Habilita o driver do sensor genérico */
CONFIG_SENSOR=y

/* Habilita a API do modem para conectividade celular */
CONFIG_MODEM=y

/* Habilita o subsistema de GNSS para GPS */
CONFIG_GNSS=y

/* Configura o nível de log para ver mensagens de depuração */
CONFIG_LOG=y
CONFIG_LOG_DEFAULT_LEVEL=4

Neste exemplo, cada linha com CONFIG_ é uma instrução que o sistema de compilação lê para incluir o código correspondente no firmware final.

Em resumo, o prj.conf é a sua ferramenta para controlar quais funcionalidades de software serão compiladas em sua aplicação, permitindo que você crie um firmware otimizado e sob medida para o seu dispositivo.
* ***VERSION***
O arquivo VERSION é uma forma simples e eficiente de gerenciar a versão do seu firmware. Ele não contém código, mas sim informações de texto que identificam a versão atual do seu aplicativo.

Pense no arquivo VERSION como a identidade do seu firmware. Ele fornece um número único que define exatamente qual código está rodando no seu dispositivo.

Qual a funcionalidade do arquivo VERSION?
A principal função do arquivo VERSION é centralizar as informações de versão para que você não precise alterá-las em vários lugares do seu código.

Ele geralmente contém campos como:

* VERSION_MAJOR: O número da versão principal (por exemplo, 1).

* VERSION_MINOR: O número da versão secundária (por exemplo, 2).

* VERSION_PATCH: O número da versão de correção de bugs (por exemplo, 3).

* VERSION_TWEAK: Um número de compilação ou revisão (por exemplo, 4).

Então, se seu arquivo VERSION contiver:

VERSION_MAJOR = 1

VERSION_MINOR = 2

VERSION_PATCH = 3

VERSION_TWEAK = 4

A versão completa do seu firmware será 1.2.3.4.

Como o Zephyr usa esse arquivo?
O sistema de compilação do Zephyr lê este arquivo e usa essas informações de várias maneiras:

* Geração Automática de Headers: O Zephyr pode usar os dados do arquivo VERSION para gerar um arquivo de cabeçalho (.h) automaticamente. Você pode então incluir esse arquivo no seu código C para que seu aplicativo saiba sua própria versão. Isso é útil para exibir a versão em um log ou em uma interface de usuário.

* Gerenciamento do Ciclo de Vida: Em projetos de IoT, a versão é fundamental. Se você tiver mil dispositivos em campo, o número da versão permite que você identifique qual firmware cada dispositivo está usando, facilitando o gerenciamento de atualizações.

* Assinatura de Imagens de Firmware: Em sistemas seguros, as imagens de firmware são assinadas criptograficamente para garantir que não sejam alteradas. A versão é um componente-chave dessa assinatura, garantindo que a imagem seja válida e que você saiba qual versão do firmware está sendo instalada.

* Em resumo, o arquivo VERSION é uma forma padronizada e automatizada de registrar o progresso do seu projeto. Ele é uma parte fundamental da gestão profissional do firmware, permitindo rastrear, auditar e gerenciar as versões do seu software de forma eficiente.

* ***main.c***: Um arquivo de código-fonte. Os aplicativos geralmente contêm arquivos de origem escrito em C, C++ ou linguagem assembly. A convenção Zephyr é colocar eles em um subdiretório de nomes .<app>src

Depois que um aplicativo for definido, você usará o CMake para gerar uma compilação diretório, que contém os arquivos necessários para criar o aplicativo e Zephyr, então vincule-os em um binário final que você pode executar em sua placa. A maneira mais fácil de fazer isso é com a construção oeste, mas você também pode usar o CMake diretamente. Os artefatos de compilação do aplicativo são sempre gerados em um diretório de compilação separado: O Zephyr não oferece suporte a compilações "na árvore".

### Criando um aplicativo

No Zephyr, você pode usar um aplicativo de espaço de trabalho de referência ou criar seu aplicativo manualmente.

#### Usando um aplicativo de espaço de trabalho de referência
O repositório Git de aplicativo de exemplo contém um workspace de referência aplicação. Recomenda-se usá-lo como referência ao criar seu próprio aplicativo, conforme descrito nas seções a seguir.

### Repositório de Aplicativo de Exemplo

A maneira mais fácil de começar a usar o repositório de aplicativos de exemplo em um espaço de trabalho Zephyr existente deve seguir estas etapas:

cd <home>/zephyrproject
git clone https://github.com/zephyrproject-rtos/example-application my-app
O nome do diretório acima é arbitrário: altere-o conforme necessário. Você agora pode entrar neste diretório e adaptar seu conteúdo para atender às suas necessidades. Desde você está usando um espaço de trabalho Zephyr existente, você pode usar ou qualquer outros comandos ocidentais para compilar, atualizar e depurar.my-appwest build

Exemplo avançado de uso de aplicativos
Você também pode usar o repositório de aplicativos de exemplo como ponto de partida para construindo sua própria distribuição de software personalizada baseada em Zephyr. Isso permite que você Faça coisas como:

remova os módulos Zephyr que você não precisa

adicionar repositórios personalizados adicionais

substituir repositórios fornecidos pelo Zephyr por suas próprias versões

Compartilhe os resultados com outras pessoas e colabore ainda mais

O repositório de aplicativo de exemplo contém um arquivo e é portanto, também um repositório de manifesto oeste. Use isso para Crie um novo espaço de trabalho personalizado seguindo estas etapas:west.yml

cd <home>
mkdir my-workspace
cd my-workspace
git clone https://github.com/zephyrproject-rtos/example-application my-manifest-repo
west init -l my-manifest-repo

Isso criará um novo workspace com a topologia T2, com como repositório de manifesto. Os nomes e são arbitrários: altere-os conforme necessário.my-manifest-repomy-workspacemy-manifest-repo

### Variáveis importantes do sistema de compilação
Você pode controlar o sistema de compilação do Zephyr usando muitas variáveis. Este descreve os mais importantes que todo desenvolvedor Zephyr deveria saber.

#### As variáveis BOARD, CONF_FILE e DTC_OVERLAY_FILE podem ser fornecidas ao sistema de compilação em 3 maneiras (em ordem de precedência):

* Como um parâmetro para a invocação ou por meio da opção de linha de comando. Se você tiver vários arquivos de sobreposição, você deve usar aspas, west buildcmake-D"file1.overlay;file2.overlay"

* Como variáveis de ambiente.

* Como uma declaração em seu set(<VARIABLE> <VALUE>)CMakeLists.txt

***Para ver as outras variáveis olhar a documentação***

***Mais especificações sobre criação de uma aplicativo neste tópico da documentação***

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

#### Depuração

Depuração, ou debugging, é o processo de encontrar e corrigir erros (bugs) no código de um programa.

A depuração de ambiente se refere a encontrar e resolver problemas que não estão no seu código em si, mas sim nas ferramentas e configurações que você usa para construir e executar seu projeto. É um tipo de depuração mais amplo que foca em todo o ecossistema do seu projeto.

No contexto de projetos como o Zephyr, a depuração de ambiente é particularmente importante, pois há muitas ferramentas e variáveis que precisam estar corretas para que o projeto funcione.

Exemplos de problemas de depuração de ambiente:

* Variáveis de ambiente incorretas: A variável ZEPHYR_BASE está apontando para o lugar errado, então o sistema de compilação não consegue encontrar os arquivos do Zephyr.

* Problemas de toolchain: O compilador (toolchain) não está instalado corretamente ou a versão é incompatível com o seu projeto.

* Hardware não detectado: A sua ferramenta de flash (como o J-Link) não consegue se comunicar com a sua placa de desenvolvimento.

* Configuração de arquivos: O seu arquivo prj.conf tem uma opção Kconfig faltando, o que faz com que o compilador não inclua um driver necessário.

É necessário uma imagem binária ELF para fins de depuração.

Uma imagem binária ELF é o arquivo executável que o compilador gera a partir do seu código-fonte. Ele contém todas as instruções, dados e metadados que o processador precisa para executar seu programa.

O nome ELF significa Executable and Linkable Format (Formato Executável e Vinculável), e é um padrão comum em sistemas Unix-like (como Linux) e também em sistemas embarcados.

#### O que uma imagem ELF contém?

Pense em um arquivo ELF como um recipiente que organiza todas as partes do seu programa:

* Código Compilado: As instruções do seu programa em linguagem de máquina, prontas para serem executadas pelo processador.

* Dados: As variáveis globais e estáticas que o seu programa usa.

* Símbolos de Depuração: Esta é a parte mais importante para o seu caso. O arquivo ELF inclui informações extras que não são necessárias para o programa rodar, mas são essenciais para a depuração.

* Mapeamento de Funções e Variáveis: O ELF "lembra" onde cada função e variável do seu código-fonte está no código compilado.

* Informações de Linha: Ele mapeia cada instrução do código compilado de volta para a linha correspondente no seu arquivo .c original.

#### Por que você precisa de uma imagem ELF para depuração?
Quando você depura um programa, você não está interagindo diretamente com o firmware que foi gravado no chip. Você usa um debugger (como o GDB) que se comunica com o chip e o arquivo ELF no seu computador.

A funcionalidade do ELF é a seguinte:

O firmware gravado no chip é uma versão "limpa" do seu código, sem as informações de depuração para economizar espaço.

O debugger no seu computador usa o arquivo ELF para saber exatamente qual linha do seu código C corresponde a qual instrução que está sendo executada no chip.

### Servidor GDB


-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

* main.c: Este é o ponto de entrada do seu código. É onde a execução do seu programa começa. A maior parte do seu código principal, como a inicialização e o loop de execução, vai aqui.

* prj.conf: Este arquivo é para configuração, não para código-fonte. Você vai colocar aqui as linhas que habilitam funcionalidades do Zephyr. Por exemplo, CONFIG_SENSOR=y para usar a API de sensores, ou CONFIG_BT=y para Bluetooth.

Outros arquivos C (.c): Para um projeto profissional, é uma boa prática separar seu código em arquivos diferentes para manter a organização. Por exemplo, você pode ter um arquivo sensor_manager.c para todo o código que lida com a leitura de sensores e um comms_handler.c para o código de comunicação. Todos esses arquivos, incluindo o main.c, são listados no arquivo CMakeLists.txt para serem compilados juntos.

#### Threads: O que são e como funcionam?
Pense em um aplicativo de smartphone: ele pode tocar música (uma tarefa) e ao mesmo tempo baixar um arquivo (outra tarefa). Ambas acontecem "simultaneamente". Em um sistema embarcado, as threads permitem que você faça o mesmo.

Uma thread é uma sequência de instruções que o kernel do Zephyr executa. O Zephyr é um sistema multitarefa, o que significa que ele pode rodar várias threads "ao mesmo tempo" (na verdade, ele troca muito rapidamente entre elas, dando a impressão de simultaneidade).

#### Por que usar threads?

* Organização: Você pode isolar a lógica. Uma thread cuida apenas da comunicação via modem, outra cuida da leitura de um sensor de movimento, e uma terceira gerencia o estado da bateria. Se uma thread travar, as outras podem continuar rodando.

* Eficiência: A maior parte do tempo, a CPU de um microcontrolador fica ociosa. Com threads, você pode colocar a thread "ociosa" para dormir enquanto outra thread que tem algo a fazer pode continuar executando. Isso é essencial para economizar energia.

#### Onde elas ficam?
As threads são partes do seu código, mas geralmente não estão no main.c em si. No main.c, você vai iniciar as threads. O código da thread (a função que ela vai executar) pode estar em um arquivo C separado, como o sensor_manager.c. A thread é uma função separada, e o main.c a inicializa com um comando como k_thread_create().

#### Device Tree (DT): A Descrição do Hardware Físico
Sua intuição está certa! O Device Tree é, sim, como uma descrição do circuito completo.

No mundo real: Você conecta o pino 3 da sua MCU ao pino SDA do seu sensor ADXL362. Você também conecta o pino 4 da MCU ao pino SCL do sensor.

No Device Tree: Você descreve essa conexão em um arquivo de texto. Por exemplo:

&i2c1 {
    adxl362@53 {
        compatible = "adi,adxl362";
        reg = <0x53>;
    };
};

Isso diz ao Zephyr: "Existe um dispositivo compatível com o driver 'adi,adxl362', ele está no barramento I2C número 1, e o endereço dele é 0x53."

Em vez de ter que "hard-codar" (#define I2C_BUS 1) os pinos e endereços no seu código, o Device Tree faz isso por você. É uma camada de abstração que separa as informações de hardware do seu código de software. Se você mudar o sensor para outro pino no futuro, basta alterar o Device Tree, não o seu código-fonte.

#### API: A Diferença para o Mundo Web
Sua compreensão de API em Web está correta, mas no contexto de um sistema embarcado, o conceito é um pouco mais "físico".

API (Web): É um conjunto de regras e protocolos que permite que um software (seu front-end) se comunique com outro (um servidor). A API define as URLs, os métodos HTTP (GET, POST) e o formato dos dados (JSON, XML).

API (Zephyr/Sistemas Embarcados): É um conjunto de funções, variáveis e estruturas que o kernel do Zephyr oferece para que seu código se comunique com o hardware.

Em vez de fazer uma chamada HTTP para um servidor, você faz uma chamada de função para o kernel do Zephyr. Por exemplo:

Para ligar um LED: gpio_pin_set(led_pin_dev, led_pin, 1);

Para ler dados de um sensor: sensor_sample_fetch(sensor_dev);

Essas funções são o "contrato" entre o seu código e o hardware. Elas são a interface que permite que você acesse as funcionalidades do hardware sem ter que saber exatamente como o chip funciona internamente. A documentação que você tem é justamente a lista de todas essas funções.

No mundo do Zephyr: A API é uma função C que você "chama" para pedir ao kernel para fazer algo com o hardware (como ligar um LED ou ler um sensor). A requisição é feita no próprio chip, e o resultado (por exemplo, a leitura do sensor) é retornado pela função.


-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Compreendendo o exemplo-application

## 2. Diretório boards
#### Por que essa estrutura é importante?
A estrutura boards/ mostra uma das maiores forças do Zephyr: a sua portabilidade.

Em um projeto de firmware profissional, é comum a empresa desenvolver sua própria placa de hardware. Em vez de ter que reescrever o código do zero para essa nova placa, a equipe de firmware pode simplesmente:

* Criar um diretório para a nova placa dentro de boards/vendor/ (por exemplo, boards/vendor/minha_placa/).

* Adicionar os arquivos de Device Tree (.dts, .overlay) que descrevem os pinos e componentes da placa.

* Adicionar os arquivos de configuração Kconfig (Kconfig.board) para habilitar os recursos da placa.

* Adicionar os arquivos .board.cmake para configurar as ferramentas de compilação e flash.

Depois de fazer isso, o mesmo código de aplicativo pode ser compilado para a sua placa personalizada ou para uma placa de desenvolvimento padrão (como a nRF9160 DK), apenas alterando a variável BOARD. Isso economiza tempo e esforço, e é um dos motivos pelos quais o Zephyr é uma escolha popular para o desenvolvimento de produtos comerciais.

#### A estratégia é:

Desenvolvimento Inicial (nRF9160 DK): Você constrói seu aplicativo no diretório app/. O seu código em C (main.c, sensor_manager.c, etc.), as suas configurações de software (prj.conf) e as sobreposições de hardware (app.overlay) serão otimizadas para a placa nRF9160 DK. Durante esta fase, o sistema de build do Zephyr já sabe como lidar com a placa porque o suporte a ela já vem embutido no SDK.

Transição para Placa Customizada: Quando chegar a hora de construir a sua própria placa com o chip nRF9160 e outros componentes, você não precisará reescrever o código do seu aplicativo. Em vez disso, você irá:

* Criar um novo diretório dentro de boards/vendor/ (por exemplo, boards/vendor/minha_placa/).

 Neste novo diretório, você criará os arquivos de configuração necessários para descrever sua nova placa.

#### O que você fazer no novo diretório da sua placa?
* Arquivo de Device Tree (.dts): Você criaria um arquivo (como minha_placa.dts) para descrever o hardware da sua placa, incluindo quais pinos estão conectados aos sensores, quais barramentos de comunicação (I2C, SPI) estão ativos e quais LEDs e botões foram adicionados.

* Arquivo de Kconfig (Kconfig.board): Você criaria um arquivo para definir as configurações de software específicas da sua placa, como habilitar drivers para os componentes que você escolheu.

* Arquivos de Compilação (.board.cmake): Se necessário, você criaria arquivos para configurar as ferramentas de compilação e flash para o seu novo hardware.

A beleza dessa abordagem é que seu código-fonte principal, localizado em app/, permanece intocado. Para compilar seu aplicativo para a nova placa, você só precisaria dizer ao west qual é a nova BOARD.

´´´Bash
// Para compilar para a placa nRF9160 DK
west build -b nrf9160dk_nrf9160

// Para compilar para a sua nova placa
west build -b minha_placa
´´´

### Explicação dos diretórios e arquivos pertencentes 

1. ***common/***: Este diretório contém arquivos que são compartilhados entre várias placas personalizadas. O arquivo que você mencionou, example_runner.board.cmake, é um ótimo exemplo disso.

* ***example_runner.board.cmake***: O sufixo .board.cmake indica que este arquivo é específico para a configuração do build de uma placa. No seu caso, este arquivo define um "runner" chamado example. Um runner é um script que o Zephyr usa para executar e depurar seu código em um hardware específico. Este arquivo em particular configura um "simulador de runner", o que é útil para testes. Ele não é algo que você usaria para a sua placa nRF9160 DK, mas mostra como você pode criar scripts personalizados para o seu próprio hardware.

Pense no runner como o controle remoto da sua placa.

#### O Que É Um "Runner"?
Um "runner" é um programa ou script que o sistema de compilação do Zephyr usa para interagir diretamente com o hardware da sua placa. Ele é a ferramenta que:

* O runner é o script que traduz os comandos de alto nível do west (west flash, west debug) para as instruções de baixo nível que a ferramenta de hardware (como o J-Link ou nrfjprog) entende.

* Grava (flashes) o firmware compilado na memória do microcontrolador.

* Inicia uma sessão de depuração (west debug).

* Executa seu programa, seja em hardware real ou em um simulado

Ele atua como uma camada de abstração entre o comando genérico do west (west flash) e a ferramenta de hardware específica (como o nrfjprog para as placas da Nordic, o J-Link ou o OpenOCD).

Para placas customizadas, você precisa criar as definições do runner manualmente. Isso é feito nos arquivos .board.cmake dentro da estrutura do diretório boards/. Essa é a forma profissional de garantir que o seu aplicativo, que é portátil, possa ser facilmente compilado, gravado e depurado em qualquer hardware que você projetar.

#### A Funcionalidade do example_runner.board.cmake
O arquivo example_runner.board.cmake que você viu no repositório example-application é um exemplo de como um desenvolvedor cria um runner personalizado. Ele faz a ponte entre o comando de compilação do Zephyr e um "runner" chamado example.

board_set_sim_runner_ifnset(example): Esta linha é a mais importante. Ela diz ao sistema de compilação do Zephyr: "Se nenhum runner já foi definido, configure a ferramenta example como o runner de simulação para esta placa."

O prefixo sim significa que ele está sendo configurado para um simulador, o que é um caso de uso comum em testes automatizados.

O nome example é apenas um nome simbólico para o script ou ferramenta que será usada.

2.  ***vendor/***: Este diretório é usado para organizar placas por fabricante (vendor). É o lugar para arquivos específicos de uma placa. É uma convenção de nomenclatura comum. Se você estivesse desenvolvendo uma placa para a Nordic, por exemplo, o nome do diretório seria nordic. O example-application usa custom_plank para simbolizar uma placa personalizada que um desenvolvedor faria por conta própria.

* ***board.cmake***: configuração específica da placa que você está usando.

1º Definição dos Runners:
As linhas board_runner_args(...) definem os argumentos (parâmetros) que são passados para os "runners" quando você os executa.

* board_runner_args(jlink "--device=nrf52" "--speed=4000"): Esta linha informa ao sistema de build que, ao usar o runner J-Link, ele deve passar os argumentos --device=nrf52 e --speed=4000. Isso garante que a ferramenta J-Link saiba para qual chip ela está gravando e em qual velocidade.

* board_runner_args(pyocd ...): Mesma coisa, mas para o runner pyOCD. Ele define o chip de destino e a frequência de comunicação.

* board_runner_args(example ...): Esta linha define um argumento para o runner customizado chamado example, que vimos anteriormente.

2º Variáveis de Ambiente:
A linha set(OPENOCD_NRF5_SUBFAMILY "nrf52") cria uma variável local para o CMake. Ela define a subfamília do chip para o OpenOCD, uma ferramenta de depuração popular. Isso permite que outros scripts usem essa variável em vez de "hard-codar" o nome do chip.

3º Inclusão de Outros Arquivos (include):
As linhas include(...) são as mais importantes. Elas "importam" a configuração de outros arquivos .board.cmake para o arquivo atual.

* include(${ZEPHYR_BASE}/boards/common/nrfjprog.board.cmake): Esta linha adiciona o runner da ferramenta nrfjprog da Nordic.

* include(${ZEPHYR_BASE}/boards/common/jlink.board.cmake): Esta linha inclui o runner para a ferramenta J-Link.

* include(${ZEPHYR_BASE}/boards/common/pyocd.board.cmake): Inclui o runner para a ferramenta pyOCD.

* include(${ZEPHYR_BASE}/boards/common/openocd-nrf5.board.cmake): Inclui o runner para a ferramenta OpenOCD para chips nRF5.

* include(${CMAKE_CURRENT_LIST_DIR}/../../common/example_runner.board.cmake): Esta linha importa o runner customizado que você viu no diretório common/, demonstrando como uma placa específica pode usar uma configuração genérica.

* ***board.yaml***: Serve para definir as propriedades básicas e metadados da sua placa personalizada. Ele é um arquivo de configuração que fornece informações de alto nível sobre o seu hardware para o sistema de compilação e para as ferramentas do Zephyr.

O conteúdo do board.yaml é escrito no formato YAML, que é fácil de ler e entender. Ele declara informações essenciais sobre a placa.

* name: custom_plank: Define o nome da sua placa. Este é o nome que você usará com o comando west build -b custom_plank.

* vendor: vendor: Especifica o fabricante da placa. A pasta onde o arquivo board.yaml está localizado deve corresponder a essa informação. No seu caso, a placa está dentro de boards/vendor/custom_plank.

* socs:: Esta seção lista os chips (SoCs - System on a Chip) que podem ser usados com esta placa.

- name: nrf52840: Indica que esta placa usa o SoC nrf52840. Isso é crucial para o sistema de compilação, pois ele usará esta informação para incluir os arquivos de configuração e drivers corretos para esse chip.

Por que o board.yaml é importante?
O board.yaml atua como o arquivo de manifesto da sua placa. Ele é a primeira coisa que o sistema de compilação do Zephyr lê quando você especifica um nome de placa.

Em resumo, enquanto o Device Tree descreve as conexões do seu hardware, o board.yaml é a identidade da sua placa para o ecossistema do Zephyr.

* ***custom_plank_defconfig***:  É um arquivo de configuração Kconfig que define as configurações padrão para a placa personalizada.

* custom_plank_defconfig (Nível da Placa): Este arquivo define as configurações padrão para o hardware. Ele ativa funcionalidades que são intrínsecas à sua placa personalizada, como o tipo de processador (CONFIG_ARM_MPU) e as proteções de hardware. O propósito dele é garantir que qualquer aplicativo que seja compilado para a custom_plank já tenha essas configurações básicas habilitadas, sem que o desenvolvedor precise se preocupar com isso no prj.conf.

* CONFIG_ARM_MPU=y e CONFIG_HW_STACK_PROTECTION=y: Estas linhas habilitam a Unidade de Proteção de Memória (MPU) e a proteção de pilha por hardware do chip, que são funcionalidades de segurança e estabilidade. A placa foi projetada com esses recursos, então eles são ativados por padrão para qualquer projeto.

* CONFIG_SERIAL=y, CONFIG_CONSOLE=y, CONFIG_UART_CONSOLE=y: Estas linhas garantem que o console via UART esteja habilitado por padrão. Isso é uma conveniência para depuração, pois permite que o desenvolvedor veja mensagens de log logo de cara, sem ter que adicioná-las manualmente em cada novo projeto.

Em suma, o defconfig é um atalho profissional. Ele centraliza as configurações básicas e essenciais da placa em um único arquivo, deixando o prj.conf livre para as configurações específicas do seu aplicativo.

* ******:
