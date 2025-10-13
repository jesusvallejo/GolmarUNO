# GolmarUNO
Golmar Intercom UNO reverse engineer 

Trabajo sobre T-540 UNO SE (repuesto de T-740 UNO ).

|Microcontrolador| LPC1114 (https://www.nxp.com/docs/en/data-sheet/LPC111X.pdf)|
|Debug| SI |
|Modo| SWD |
|Conexion| Header, de abajo a arriba 3v3,swio,swclk,nc,nc,gnd |
|Extraccion de firmware| SI, Openocd. openocd -f 'interface/stlink.cfg' -f 'target/lpc11xx.cfg' -c "adapter speed 1000"  .  reset_init;halt;flash read_bank 0 t-540.bin |

## Comunicación entre dispositivos
Serial, mediante un hilo contectados todos al mismo hilo. 5 voltios, polaridad invertida. El LPC1114 cuenta con puertos UART estandar rx/tx compatibles tanto con 3v3 como 5v.
Para hacer compatible el micro con el bus uni-hilo, la placa cuenta con un chip LM358, y un par de transistores. Los pines conectados son el 37 (RX) y 36 (TX).
Debajo de la placa, hay dos test-points que permiten acceder facilmente a estos pines.
## Protocolo
Veremos este apartado desde el punto de vista del controlador, en el que contamos con señales UART estandar. Tras diferentes analisis se llega a la conclusion:
Baud | 2600
Stopbit | 1
Parity | even
data_bits| 8
Durante el analisis, se encontro en un foro que se identificaba como una conexion multidrop, de 9 bits, y tras un analis de la señal tambien encaja, sin embargo no hay diferencias notadas entre usar uno u otro.
Por comodidad, compatibilidad y simplicidad nos quedamos con nuestro analisis.





