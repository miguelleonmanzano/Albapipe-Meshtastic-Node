# Albapipe-Meshtastic-Node
# PÁGINA EN CONSTRUCCIÓN / PCB EN PRUEBAS #
Nodo para tecnología Meshtastic de hasta 1 w de potencia preparado para introducir en un tubo.

Albapipe es un proyecto para la construcción de un nodo con las siguientes características:
- Módulo de radio de 1 w.
- Controlador de bajo consumo y coste.
- Monitorización simultánea de 3 consumos de corriente.
- Reseteo automático por baja tensión (opcional).
- Reseteo automático por tiempo configurable físicamente (opcional).
- Monitorización de temperatura (opcional).
- GPS (opcional).
  
- El modelo en 2D es el siguiente:
  
<img width="65" height="580" alt="image" src="https://github.com/user-attachments/assets/4cd5cd09-830d-455e-9147-86bbe93bbcb5" />   <img width="65" height="579" alt="image" src="https://github.com/user-attachments/assets/59884425-bf67-4e40-90b8-ab8811a152a3" />

- El modelo en 3D quedaría como a continuación:
  
<img width="82" height="738" alt="image" src="https://github.com/user-attachments/assets/2c3a412f-103b-4c14-880a-8786cc1ae130" />                        <img width="82" height="728" alt="image" src="https://github.com/user-attachments/assets/a19dd270-b4d8-4887-952f-0f1ea349bbff" />
# FUNCIONAMIENTO BÁSICO #
- El proyecta utiliza como "cerebro" al microcontrolador Promicro. Este gestiona las funciones principales de gestión de los datos, recepción y envío de mensajes, gestión de la radio, ect.
El equipo de radio elegido es el E22P por su relación de prestaciones (amplificador, presencia de filtro, ...). Auxiliarmente se cuenta con los sistemas TLV, que desconecta (aplica un reset en el microcontrolador)
todo el sistema cuando la tensión desciende de 3.0 voltios y el microcontrolador ATTiny cuya función es resetear el equipo cada cierto tiempo (tiempo programable) ante eventuales bloqueos no esperados del
microcontrolador principal. Un controlador de carga CN3791 gestiona la corriente que proviene del panel solar a las baterías. Un BMS por batería protegen a las mismas ante sobrecarga o cortocircuito. El GPS y el módulo de radio no funcionan a pleno rendimiento, o producen malfuncioanmiento si su alimentación no es de 5 V, por lo que se implementan dos convertidores DC-DC para alimentarlos. Estos dos BOOST son apoyados con dos condensadores de 1000 uF en su entrada y salida para evitar que, en momentos de alta demanda de corriente, se produzca una caída de tensión.
# BOM #
La lista de materiales necesarios es la siguiente:
  - Componentes principales:
- PCB Albapipe:                            Descargar Gerbers y encargar fabricación a JLCPC o NextPCB.
- Microcontrolador:                        Promicro NRF52840
- Transmisor de radio:                     E22 o E22P (E22P-868M30S)
- Supervisor de corriente I2C:             INA3221
- Controlador de carga:                    CN3791
- Conector de batería y solar:             2 x PA001-2P
- Interruptores energía:                   2 x SS12D10-G5
- Pulsadores de Reset / User:              2 x TC-1101T-C-B-B  
- Boost para el E22:                       Boost DC-DC 5V.
- Opcional: Boost para el GPS:             Boost DC-DC 5V.
- Opcional: Supervisor de tensión:         TLV840MADL30DBVRQ1
- Opcional: Conector de antena:            KH-SMA-P-8496-T.
- Opcional: Reset programable.             ATTiny 13A-PU + C=100 nF + R10K
- Opcional: Telemetría ambiental:          BME/BMP280
- Opcional: Divisor de tensión:            2 x R 1M
- Opcional: GPS:                           NNEO6MV2            

  - Componentes auxiliares:
- Tubo de PVC de 40 mm.
- Tapón "cerrado" PVC para tubo de 40 mm.
- Registro "tapón enroscable" de PVC para tubo de 40 mm.
- Panel solar de 6 w / 9 w - 5 voltios.
- Sujección tubo a pared metálica.

# PROCESO DE FABRICACIÓN #
