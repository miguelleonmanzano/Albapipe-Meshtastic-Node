
# 🌞 ALbaPipe. Nodo Meshtastic Solar Autónomo y Robusto  

**Control de versiones:**
| VERSIÓN | ESTADO | FECHA |
|----:|----:|-----------|
| 1 | TESTADA ✅  | Enero 2026 |
| 2 | 🚧 EN PRUEBA 🚧  | Enero 2026 |

Este proyecto describe el diseño de un **nodo solar Meshtastic completamente autónomo**, concebido para **despliegues reales en campo**, con especial énfasis en **fiabilidad eléctrica**, **tolerancia a fallos** y **estabilidad a largo plazo**.

El diseño integra **gestión energética avanzada**, **separación de cargas**, **supervisión de tensión**, **monitorización de consumos** y un **watchdog hardware independiente**.

---

## 🧠 Enfoque y filosofía del diseño

Este no es un nodo experimental. Está pensado para funcionar **meses o años sin intervención**, incluso en condiciones desfavorables.

Principios clave:

- ✅ **Firmware Meshtastic sin modificar** (firmware oficial)
- ✅ Recuperación automática ante cuelgues mediante **hardware externo**
- ✅ Aislamiento eléctrico entre bloques críticos
- ✅ Gestión eficiente de energía solar + baterías
- ✅ Arquitectura clara, modular y mantenible

---

## 🧩 Bloques funcionales del sistema

### 🔋 Gestión energética
- **Panel solar**
- **Cargador MPPT CN3791**
- **BMS 1S**
- **Baterías Li‑ion 3,7 V en paralelo (1S3P)**
- **Selector de fuentes / conectores de alimentación**

### ⚡ Regulación de tensión
- **Salida regulada desde el NRF a 3.3 V** para lógica y sensores
- **Boost 5 V independiente #1 (HW‑085)** → Para el módulo de radio LoRa E22 / E22P
- **Boost 5 V independiente #2 (HW‑085)** → GPS. Con "enable" por GPIO para reducción de consumo

### 📡 Comunicaciones
- **Radio LoRa E22 / E22P** con salida a conectores IPEX o SMA.
- **GPS NEO** u otros compatibles.

### 🧠 Control y supervisión
- **nRF52840** ejecutando Meshtastic
- **ATtiny13A** como watchdog externo, con reset automático preconfigurable
- **Supervisor TLV840 (~3.0 V)** para protección por batería baja

### 📊 Monitorización
- **INA3221 (I²C, 3 canales)** para:
  - Corriente del panel solar
  - Corriente de carga
  - Consumo del sistema
- **BME/BMP 680** para:
  - Temperatura
  - Humedad
  - Presión barométrica
---

## 🔋 Arquitectura de alimentación

Componentes principales

- Panel Solar → Entrada fotovoltaica primaria.
- MPPT CN3791 → Control de carga optimizada para baterías Li‑ion 1S.
- Batería Li‑ion 1S3P (3.0–4.2 V) + BMS 1S → Almacenamiento y protección.
- Rail 3.3 V (Lógica) → MCU/sensores de bajo consumo.
- Boost 5 V HW‑085 #1 → Alimenta LoRa E22 / E22P
- Boost 5 V HW‑085 #2 → Alimenta GPS NEO.

### Motivación técnica

- El **E22** presenta picos importantes en transmisión
- El **GPS** es sensible a ruido y caídas de tensión
- La separación de boosts evita interferencias mutuas
- La lógica a 3.3 V queda aislada de transitorios de potencia
- De este modo se separan los railes de 5 V para aislar picos de corriente del GPS y la radio.
---

## 📡 Radio LoRa (E22‑868M30S)

- Banda **868 / 915** 
- Alimentación dedicada a **5 V**
- Antena externa SMA

---

## 🛰️ GPS

- Módulo **NEO**
- Alimentación dedicada a **5 V**
- Encendido controlado por GPIO
- Totalmente separado eléctricamente del LoRa

Permite posicionamiento y telemetría sin comprometer la estabilidad del sistema.

---

## 🔁 Watchdog hardware independiente (ATtiny13A)

El nodo integra un **watchdog físico externo**, completamente independiente del nRF52840.

### ¿Por qué es necesario?
- Un MCU no puede auto‑recuperarse si queda bloqueado
- Meshtastic es estable, pero ningún firmware es inmune
- La fiabilidad real exige **supervisión fuera del firmware**

### Funcionamiento
- El **ATtiny13A** permanece en *sleep* profundo
- Se despierta periódicamente mediante temporizador interno
- Genera un **pulso directo sobre RESET del nRF52840**


### ⏱️ Periodos seleccionables por jumpers

| PB2 | PB1 | Reset cada |
|----:|----:|-----------|
| 0 | 0 | **1 minuto** |
| 0 | 1 | **6 horas** |
| 1 | 0 | **12 horas** |
| 1 | 1 | **24 horas** |

- Pulso de reset: **200 ms (LOW)**
- Consumo ultra bajo
- Funciona incluso si el nRF está totalmente colgado siempre que cuente con una mínima tensión de alimentación.
  
<img width="224" height="203" alt="image" src="https://github.com/user-attachments/assets/a1405204-e1c1-43c1-8cbd-a2a800cf0ded" />


---

## 🛡️ Supervisor de tensión (TLV840)

- Monitoriza la tensión de batería
- Fuerza la desconexión por debajo de ~**3.0 V**, remitiendo una señal de reset al MCU.
- Evita estados inestables al descargar la batería
- Complementa al watchdog periódico


## 🛡️ Circuito divisor de tensión

- Monitoriza la tensión de batería para que el NRF pueda interpretarla en tanto por ciento.


  ---

## 📊 Monitor de corriente (INA3221)

- Bus **I²C**
- Tres canales independientes
- Permite instrumentar:
  - Rendimiento del panel
  - Eficiencia de carga
  - Consumo del MCU. 

Base ideal para **telemetría energética** y optimización.

---

## 🧩 Resumen de módulos utilizados

| Bloque | Componente |
|------|-----------|
| MCU principal | nRF52840 | Obligatorio |
| Watchdog | ATtiny13A | Opcional |
| LoRa | E22‑868M30S - E22P‑868M30S | Obligatorio |
| GPS | NEO | Opcional |
| MPPT | CN3791 | Obligatorio |
| Boost 5 V | HW‑085 (×2) | Obligatorio |
| Supervisor | TLV840 | Opcional |
| Monitor | INA3221 | Opcional |
| Baterías | Li‑ion 1S3P | Obligatorio (una al menos) |
| Protección | BMS 1S | Opcional |
| RF | SMA + antena | Opcional |
| Divisor de tensión | 1MOhm (x2) | Opcional |
---

## 📦 Bill of Materials (BOM) — Nodo Meshtastic Solar


| ID | Nombre | Modelo | Cantidad |
|----|----------------------------------------------|----------------------------------------------|----------|
| 0  | PCB ALBAPIPE V1 | ENCARGAR EN NEXTPCB o JLCPB | 1 |
| 1  | SOPORTES BATTERY2, BATTERY3, BATTERY1 | BH-18650 | 3 |
| 2  | SUPERVISOR CORRIENTE | INA3221 | 1 |
| 3  | E22_BOOST | DC_DC_BOOST1 | 1 |
| 4  | GPS_BOOST | DC_DC_BOOST2 | 1 |
| 5  | BMS IC3, IC2, IC1 | BMS 1S 3.7V | 3 |
| 6  | CONECTORES BATERÍA Y SOLAR | PA001-2P | 2 |
| 7  | BOTONES USER Y RESET | TC-1101T-C-B-B | 2 |
| 8  | GPS ENABLE / ATTINY RESET R3, R2 | 10K | 2 |
| 9  | WATCHDOG TIMER U1 | ATTINY13A-PU | 1 |
| 10 | ATTINY RESET U2 | 100 nF | 1 |
| 11 | GPS ENABLE Q1 | SI2312 | 1 |
| 12 | GPS U6 | GPS NEO6MV2 | 1 |
| 13 | SUPERVISOR DE TENSIÓN U5 | TLV840 | 1 |
| 14 | INTERRUPTORES MCU Y SOLAR | SS12D10-ZG5 SWITCH | 2 |
| 15 | MÓDULO BOOST C1, C2 | 1000uF | 2 |
| 16 | CONECTOR ANTENA | KH-SMA-P-8496-T | 1 |
| 17 | B+, B- | 1M | 2 |
| 18 | TELEMETRÍA AMBIENTAL | BME280 | 1 |
| 19 | CARGADOR DE BATERÍAS | CN3791 MPPT Solar Charger Module | 1 |
| 20 | TRANSMISOR LORA E22 / E22P | E22P-868M30S (UE) | 1 |
| 21 | MCU | PRO_MICRO_NRF52840 | 1 |

Todas las resistencias y condensadores SMD son de tipo 1206..

---

### 🧱 Mecánica
| Ref | Componente | Modelo / Valor | Qty |
|---|---|---|---:|
| — | PCB | FR‑4 | 1 |
| — | Encapsulado | Tubo PVC Ø50 mm, tapones fijos y enroscables | 1 |


## 🧱 Diseño preparado para encapsulado en tubo de PVC Ø50 mm

El nodo ha sido concebido desde el inicio para poder **introducirse en un tubo de PVC de fontanería de 50 mm de diámetro**, un formato muy utilizado en instalaciones de campo por su **robustez, disponibilidad y bajo coste**.

### Justificación técnica

- La **disposición lineal de los módulos** (baterías 1S3P, electrónica y radio) permite un **form factor alargado**, compatible con tubos estándar de Ø50 mm.
- El uso de **módulos compactos** (E22, HW‑085, ATtiny13A, INA3221) y la ausencia de elementos voluminosos facilita el encapsulado cilíndrico.
- La **antena externa SMA** puede sacarse axialmente por uno de los extremos sin comprometer la estanqueidad.
- El diseño no depende de ventilación activa, lo que favorece un **encapsulado completamente sellado**, aunque se recomienda instalar un tapón de ventilación para evitar sobrepresión o vacío.

### Ventajas del encapsulado en tubo de PVC

- ✅ **Alta resistencia mecánica** frente a golpes, vibraciones y fauna
- ✅ **Excelente comportamiento frente a humedad, polvo y lluvia**
- ✅ Fácil **sellado con tapones estándar** o racores
- ✅ Integración sencilla en **postes, mástiles o enterrado parcial**
- ✅ Coste muy bajo y materiales disponibles en cualquier ferretería
- ✅ Discreción visual en entornos naturales o rurales

---



##  FABRICACIÓN

##  IMPORTANTE ANTES DE COMENZAR.

- El sistema no está protegido ante inversión de polaridad de baterías. Tener la máxima precaución para respetar la polaridad impresa en la placa y los soportes.
- NO alimentar la PCB sin instalar una antena en el módulo PCB.
- Ajustar la temperatura del soldador en función del componente. Por ejemplo, las conexiones a los soportes de batería requerirán mucha más temperatura que los componentes SMD o los pad de pequeño tamaño.

##  Proceso de fabricación de la PCB.

- Soldar en este orden:
- Componentes SMD.
- Componentes de menor tamaño.
- Resto de módulos.
- Conectores.
- Soportes de baterías.
- Se recomienda instalar cinta aislante entre el E22 / E22P y la placa, para evitar contactos involuntarios con las conexiones a la placa de la otra cara de la misma.
  
<img width="322" height="220" alt="image" src="https://github.com/user-attachments/assets/7bee67bf-2da5-4c15-8cde-9b2e18ca3c4e" />

- Se recomienda instalar el MCU sobre un zócalo. Esto permite su reemplazo (por avería, actualización, ...) de manera más sencilla:

<img width="174" height="331" alt="image" src="https://github.com/user-attachments/assets/8c9ab748-9ad4-4767-9609-ebd88a0f688e" />

- El modelo en 2D es el siguiente:
  
<img width="65" height="580" alt="image" src="https://github.com/user-attachments/assets/4cd5cd09-830d-455e-9147-86bbe93bbcb5" />   <img width="65" height="579" alt="image" src="https://github.com/user-attachments/assets/59884425-bf67-4e40-90b8-ab8811a152a3" />

- El modelo en 3D quedaría como a continuación:
  
<img width="82" height="738" alt="image" src="https://github.com/user-attachments/assets/2c3a412f-103b-4c14-880a-8786cc1ae130" />                        <img width="82" height="728" alt="image" src="https://github.com/user-attachments/assets/a19dd270-b4d8-4887-952f-0f1ea349bbff" />

##  Errores de diseño NO CRÍTICOS. La versión V2 (en pruebas) no tendrá estos problemas.

- 1 Error en footprint del BME280. Hay que forzar los pines:
<img width="132" height="106" alt="image" src="https://github.com/user-attachments/assets/006df5d3-2434-44e7-9d76-ecce2ab860a4" />

- 2 Error en footprint de los Boost: Hay que forzar los pines:
<img width="227" height="185" alt="image" src="https://github.com/user-attachments/assets/83b7fcc4-5762-43ad-b387-3450cd85b1cb" />

               

- Primer modelo fabricado. El acabado no es el mejor al realizarle mcuhas pruebas:

<img width="1129" height="235" alt="image" src="https://github.com/user-attachments/assets/a6f4d59a-29a9-4952-9f58-0cf3ba225eb3" />

<img width="1139" height="173" alt="image" src="https://github.com/user-attachments/assets/43170cff-4d78-4690-8fdd-6015380d292a" />


##  Ejemplo de instalación en tubo de 50 mm.

<img width="341" height="638" alt="image" src="https://github.com/user-attachments/assets/2e7d3817-1c5f-4ce5-9f61-8dcdf8feee40" />

<img width="550" height="300" alt="image" src="https://github.com/user-attachments/assets/8e5ded0c-d819-44a9-bd06-97684f2c9192" />


## 📜 Licencia
Ver archivo "License" adjunto.


