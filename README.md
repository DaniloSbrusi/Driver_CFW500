# Driver Modbus TCP para inversor WEG CFW500

Driver de comunicação Modbus TCP de CLP Siemens com inversor de frequência WEG CFW500 com entradas e saídas pré-definidas em bloco único encapsulado em biblioteca para controle e monitoramento do acionamento de um motor. Desenvolvido no Tia Portal da Siemens como tarefa da disciplina de "Tópicos especiais em CLP e redes" do curso de Engenharia de Controle e Automação do IFC Campus Luzerna.

### Sobre o Modbus TCP

O protocolo de comunicação Modbus TCP permite uma troca de dados segura entre diversos tipos de dispositivos com alta velocidade. Em relação ao acesso, segue o modelo **cliente-servidor**. O **cliente** é responsável por iniciar as requisições — enviando comandos de leitura ou escrita — enquanto o **servidor** apenas responde às solicitações recebidas. Nesse driver, o **CLP Siemens S7-1200** atua como **cliente Modbus TCP**, enviando comandos através do bloco de função **MB_CLIENT**, e o **inversor WEG CFW500** atua como **servidor**, respondendo às requisições conforme sua tabela interna de registradores.  

A variante TCP do Modbus utiliza o meio físico Ethernet. O endereçamento dos dispositivos é feito pelo protocolo IP e a transação de dados utiliza a porta 502 do TCP. Cada mensagem contém um **cabeçalho Modbus Application Protocol (MBAP)**, que inclui informações como o identificador da transação, identificador do protocolo, comprimento e o identificador da unidade. O conteúdo do telegrama ainda contém um **código de função** e os **dados** (endereços e valores dos registradores).   

O driver construído utiliza o bloco "MB_Client" do Tia Portal que é capaz de estabelecer uma conexão Modbus TCP por meio da interface PROFINET da CPU do CLP Siemens. São utilizados os códigos de função Modbus 3 e 16, para leitura e escrita de "holding registers" abstraídos para os modos 0 e 1, respectivamente.  

## Configuração do inversor

Antes de iniciar a configuração do driver, você deve:
- Instalar o módulo de comunicação ethernet (ou profinet) do inversor 
- Conectar, via ethernet, o inversor e o CLP na mesma rede
- Energizar os equipamentos

Configurar os parâmetros básicos do inversor (consulte manual de programação do CFW500)  
Usando, por exemplo, um motor trifásico 380V, 1.1A, 1700rpm, 60Hz, 0.37kW e 0.71 FP

| Configuração                                             | Parâmetro | Valor | Descrição                                        |
| -------------------------------------------------------- | --------- | ----- | ------------------------------------------------ |
| Carregar configurações de fábrica do inversor (opcional) | 204       | 5     | Carrega padrão 60 Hz                             |
| Tensão nominal do motor                                  | 400       | 380   | Exemplo, motor 380 V                             |
| Corrente nominal do motor                                | 401       | 1.1   | Exemplo, motor 1,1 A                             |
| Velocidade nominal do motor                              | 402       | 1700  | Exemplo, motor 1700 rpm                          |
| Frequência nominal do motor                              | 403       | 60    | Exemplo, motor 60 Hz                             |
| Potência nominal do motor                                | 404       | 3     | Exemplo, motor 0,37 kW                           |
| Fator de potência nominal do motor                       | 407       | 0.71  | Exemplo, motor 0,71 FP                           |
| Seleção de situação local/remoto                         | 220       | 2     | Seleção via teclas da IHM                        |
| Seleção de referência de velocidade na situação local    | 221       | 0     | Seleção via teclas da IHM                        |
| Seleção de referência de velocidade na situação remoto   | 222       | 11    | Seleção via Ethernet (Modbus TCP)                |
| Seleção do sentido de giro na situação local             | 223       | 2     | Seleção via teclas da IHM                        |
| Seleção do gira/para na situação local                   | 224       | 0     | Seleção via teclas da IHM                        |
| Seleção de JOG na situação local                         | 225       | 1     | Seleção via teclas da IHM                        |
| Seleção do sentido de giro na situação remoto            | 226       | 9     | Seleção via Ethernet (Modbus TCP)                |
| Seleção do gira/para na situação remoto                  | 227       | 4     | Seleção via Ethernet (Modbus TCP)                |
| Seleção de JOG na situação remoto                        | 228       | 5     | Seleção via Ethernet (Modbus TCP)                |
| Configuração do endereço IP (conforme rede)              | 810       | 0     | Definição manual, sem DHCP                       |
| Endereço IP (primeiro byte)                              | 811       | 172   | Exemplo, endereço 172.16.80.30                   |
| Endereço IP (segundo byte)                               | 812       | 16    | Exemplo, endereço 172.16.80.30                   |
| Endereço IP (terceiro byte)                              | 813       | 80    | Exemplo, endereço 172.16.80.30                   |
| Endereço IP (quarto byte)                                | 814       | 30    | Exemplo, endereço 172.16.80.30                   |
| Máscara de rede                                          | 815       | 24    | Conforme tabela, configura máscara 255.255.255.0 |
| Endereço gateway (primeiro byte)                         | 816       | 172   | Exemplo, endereço 172.16.80.1                    |
| Endereço gateway (segundo byte)                          | 817       | 16    | Exemplo, endereço 172.16.80.1                    |
| Endereço gateway (terceiro byte)                         | 818       | 80    | Exemplo, endereço 172.16.80.1                    |
| Endereço gateway (quarto byte)                           | 819       | 1     | Exemplo, endereço 172.16.80.1                    |
| Atualização configurações Ethernet                       | 849       | 1     | Equivalente a reiniciar o inversor               |

## Configuração do driver:

- Faça download dos arquivos
- No Tia Portal, após configurar a CPU, na aba "Libraries" -> "Global Libraries" -> clique no botão "Open Global Library"
- Selecione a pasta que você acabou de baixar e abra o arquivo "Driver_CFW500.al17"
- Na seção de "Global Libraries", localize o item "Driver_CFW500"
- Em "Master copies" -> "Driver data types", selecione os itens e arraste para a seção "PLC data types" da sua CPU
- Em "Master copies" -> "Driver blocks", selecione os blocos e arraste para a seção "Program blocks" da sua CPU
- Para configuração do driver, crie um novo DB em "Program blocks" -> "Add new block"
- Como nome do novo DB, digite "configuracao" e selecione o tipo "driver_config" para criar a estrutura correta
- No bloco "configuracao" insira apenas o endereço IP em "Static" -> "MB_client" -> "RemoteAddress" -> "ADDR" conforme configurado no inversor 
- Para possibilitar a leitura ou escrita de qualquer parâmetro do inversor, crie também o DB "comunicacao" do tipo "driver_comunic"
- No seu projeto, em "Main", por exemplo, inclua apenas o bloco "Driver CFW500", criando o DB associado a sua instância

---

## Descrição das entradas e saídas do bloco "Driver_CFW500"

### "ligar"
Entrada do tipo booleano capaz partir e parar o motor  
"TRUE" -> Dá a partida no motor utilizando a rampa de aceleração definida (escreve o valor 0b00000011 no parâmetro P0684 do inversor)  
"FALSE" -> Para o motor utilizando da rampa de desaceleração definida (escreve o valor 0b00000010 no parâmetro P0684 do inversor)
### "ref_velocidade"  
Entrada do tipo UInt que ajusta a referência de velocidade do inversor (escreve o valor da entrada no parâmetro P0685 do inversor)  
Para programar a referência de velocidade, consulte o manual da interface Ethernet do inversor WEG CFW500  
Exemplo: se a frequência nominal (P0403) for 60Hz, então a velocidade de 60Hz é obtida entrando com o valor "8192"
### "rampa_aceleracao"  
Entrada do tipo UInt que ajusta o tempo da rampa de aceleração do inversor (escreve o valor da entrada no parâmetro P0100 do inversor)  
Para programar a rampa de aceleracao, consulte o manual de programacao do inversor WEG CFW500  
Exemplo: se a rampa desejada é de 4.5 segundos, insira na entrada o valor "45"
### "rampa_desaceleracao"  
Entrada do tipo UInt que ajusta o tempo da rampa de desaceleração do inversor (escreve o valor da entrada no parâmetro P0101 do inversor)  
Para programar a rampa de desaceleracao, consulte o manual de programacao do inversor WEG CFW500  
Exemplo: se a rampa desejada é de 8.0 segundos, insira na entrada o valor "80"
### "config"
Entrada do tipo "driver_config"
Inclua o DB chamado "configuracao" e nele ajuste o endereco IP do inversor CFW500
### "comunic"
Entrada e saída do tipo "driver_comunic" utilizada para leitura e escrita dos parâmetros do inversor CFW500
Inclua o DB chamado "comunicacao"
Para mais detalhes sobre a leitura e escrita de parâmetros, consulte a seção de "Leitura e escrita de parâmetros do inversor"
### "velocidade"
Saída do tipo Word que indica a velocidade atual do motor em hexadecimal (leitura do parâmetro P0002)
### "corrente"
Saída do tipo Word que indica a corrente elétrica do motor em hexadecimal (leitura do parâmetro P0003)
### "falha"
Saída do tipo Word que indica o número da falha atual que eventualmente esteja presente no inversor (leitura do parâmetro P0049)

---
## Leitura e escrita de parâmetros do inversor 

A troca de dados com o inversor se dá pelo DB "comunicacao" do tipo "driver_comunic"
Nesse DB existem 5 estruturas (array) utilizadas pelos 4 canais de comunicação:

- "**parametros**": entrada do tipo UInt em que podem ser incluídos até 4 endereços de parâmetros do CFW500
- "**leitura_escrita**": entrada do tipo BOOL que define o modo de comunicacao para cada um dos 4 parâmetros individualmente
    "FALSE": indica modo de LEITURA do parâmetro 
    "TRUE": indica modo de ESCRITA no parâmetro 
- "**dado_leitura**": saída do tipo WORD (valor que será lido do parâmetro)
- "**dado_escrita**": entrada do tipo WORD (valor que será escrito no parâmetro)
- "**trigger**": entrada do tipo BOOL que aciona a leitura ou escrita do parâmetro com pulso TRUE, conforme configurado em "leitura_escrita"


> Atenção!
> A entrada de trigger deve receber apenas um **pulso** TRUE.   
> Manter o trigger ativo por muito tempo pode comprometer o funcionamento do driver, uma vez que serão solicitas diversas leituras/escritas sequencialmente.

*Exemplos de comunicação:*  

Para ler o parâmetro P0100 (tempo da rampa de aceleração) usando o canal 0 de comunicação do driver
- em parametros[0] insira o valor "100"
- em leitura_escrita[0] insira o valor "FALSE"
- em trigger[0] dê um pulso de valor "TRUE"
O valor do parâmetro P0100 será entregue em dado_leitura[0]

Para escrever o valor 0 no parâmetro P0216 (desliga a iluminação do display do inversor) usando o canal 2 de comunicação do driver
- em parametros[2] insira o valor "216"
- em leitura_escrita[2] insira o valor "TRUE"
- em trigger[2] dê um pulso de valor "TRUE"
O valor 0 será escrito no parâmetro P0216
