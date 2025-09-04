# nRF_conectado_ao_VSCode
Aprendendo sobre como utilizar o nRF no VS Code

* nRF Connect SDK: Contém o Zephyr RTOS e as bibliotecas da Nordic. É a parte principal do seu ambiente de desenvolvimento. Se algum arquivo estiver corrompido, a reinstalação resolve.

* nRF Connect SDK Toolchain: É o compilador (GCC) e o conjunto de ferramentas que transformam seu código em um arquivo executável. Uma instalação corrompida na toolchain é uma causa comum de erros de compilação.

* nRF Command Line Tools: Contém apenas ferramentas de linha de comando para interagir com a placa (fazer o flash, apagar, etc.). Como você não está usando uma placa física, esse pacote não está envolvido na falha de compilação para native_sim ou QEMU.

## Criação de uma aplicação:
1. Criar uma configuração de construção:
Informa ao compilador quais arquivos da placa incluir na construção, tornando a saída compatível com a placa.
3. 
