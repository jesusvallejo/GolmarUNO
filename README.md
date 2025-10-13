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

### Formato de comunicación
 T-540
  Los dispositivos se comunican haciendo uso de mensajes de 4 bytes. Se ha observado que los tres primeros hacen referencia a la dirección y el último al comando.
  Ej: 00 00 12 01 - Dispositivo 12 Comando 01
  Normalmente la comunicación siempre se inicia por parte de la placa principal hacia los telefonillos, a excepción del terminal del conserje.
  La comunicación se inicia en el momento en el que se presiona el pulsador de la placa, y esta envia el comando 37, ej: 00 00 13 37 . 
  En ese momento el telefonillo con la direccion 13 responde con el comando ACK 01, ej: 00 00 13 01.
  La placa principal no tiene dirección, y la comunicacion siempre se hace con referencia al telefonillo al que se llama.
  Una vez descolgado el telefonillo, este informa a la placa de iniciar el enrutamiento del audio, comando 10, ej: 00 00 13 10, seguido de un comando que limpia el bus , en este caso el comando es general y no tiene dirección : 00 00 00 11
  La placa devuelve un ACK 01, ej: 00 00 13 01 , y se inicia la comunicación.
  En caso de accionar el boton de abrir se envia el comando 90: 00 00 13 90 , este comando solo funciona si hay una llamada en curso, no se puede engañar a la placa principal.
  CE-941
  
  
  



