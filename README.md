# GolmarUNO
Golmar Intercom UNO reverse engineer 

Trabajo sobre T-540 UNO SE (repuesto de T-740 UNO ).

Microcontrolador: LPC1114 (https://www.nxp.com/docs/en/data-sheet/LPC111X.pdf)
Debug: SI
Modo: SWD
Conexion: Header, de abajo a arriba 3v3,swio,swclk,nc,nc,gnd
Extraccion de firmware: SI, Openocd. openocd -f 'interface/stlink.cfg' -f 'target/lpc11xx.cfg' -c "adapter speed 1000"  .  reset_init;halt;flash read_bank 0 t-540.bin



