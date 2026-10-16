<h1 align="center">Matías López</h1>

<p align="center">
  <b>Estudiante de Ingeniería en Computación — UNLP · La Plata, Argentina</b><br/>
  Ciencia de datos aplicada · Robótica y sistemas embebidos · Arquitectura y diseño digital
</p>

<p align="center">
  <a href="https://www.linkedin.com/in/ingmatiaslopez/">
    <img src="https://img.shields.io/badge/LinkedIn-ingmatiaslopez-0A66C2?logo=linkedin&logoColor=white" alt="LinkedIn"/>
  </a>
  <img src="https://img.shields.io/badge/Legajo-02670%2F0-555" alt="Legajo"/>
</p>

---

Trabajo sobre problemas donde el dato y el hardware se tocan: modelos que predicen una
magnitud física, firmware que la mide, y la electrónica que sostiene a los dos. Mi práctica
profesional fue un modelo de pronóstico solar para el **LEICI (UNLP–CONICET)**, y en paralelo
curso Taller de Proyecto I construyendo un brazo robótico sobre EDU-CIAA-NXP.

Lo que intento que se note en cada repo: **decisiones justificadas con números**, no con
preferencias. Cada proyecto grande lleva su registro de decisiones, su documentación y sus
resultados contra un baseline.

---

## Ciencia de datos y aprendizaje automático

Es el área donde más profundizo. Combina lo que más me interesa: modelar un fenómeno real,
medir si el modelo sirve de verdad, y poder explicar por qué funciona.

### Pronóstico de irradiancia solar con XGBoost — Práctica Profesional Supervisada

**[`PPS-Entrega`](https://github.com/MatiasLopezING/PPS-Entrega)** · LEICI · Facultad de
Ingeniería + Facultad de Informática, UNLP

Predicción de la irradiancia global horizontal (GHI) a seis horizontes simultáneos —de 10 a
60 minutos— sobre siete años de datos del **NSRDB** para La Plata, con resolución de diez
minutos. Evaluado sobre 2023–2024, período no usado en ninguna etapa de ajuste, contra el
**baseline de persistencia** que es la referencia estándar del campo.

| Métrica (promedio de los 6 horizontes) | Valor |
|---|---|
| R² | 0,8845 |
| MAE | 62,75 W/m² |
| RMSE | 102,28 W/m² |

Supera a la persistencia en todos los horizontes, y la ventaja **crece con la distancia
temporal**: 9,1 % de mejora a 10 minutos, 33,3 % a 60. Entre t+1 y t+6 su error aumenta un
51,8 % frente al 106,9 % del baseline — se degrada a la mitad del ritmo.

Lo que me parece más valioso del trabajo no es el número final sino las decisiones:

- **Prevención de fuga temporal.** Solo entran como información futura las magnitudes
  astronómicas y de calendario, computables con anticipación arbitraria. Esto excluyó
  deliberadamente a `Clearsky GHI`, que el NSRDB calcula con reanálisis no disponible en
  tiempo real y que habría inflado los resultados.
- **Estrategia directa multi-salida**: seis regresores independientes en lugar de
  reinyección recursiva, para no acumular error a lo largo de la cadena.
- **Selección de ventana por parsimonia**: cinco ventanas de memoria evaluadas con validación
  cruzada temporal. Diferencias menores al 1 % y curva no monótona ⇒ configuraciones
  estadísticamente equivalentes. Se adoptó la de 78 variables de entrada en lugar de 414, con
  el mismo error.
- **Interpretabilidad con SHAP**: el ordenamiento de variables resultó coherente con la física
  del problema, lo que confirma que el modelo combina geometría solar determinista con estado
  atmosférico reciente en lugar de memorizar.
- **Atípicos tratados según su naturaleza** (Fox, 1972): los picos aislados se interpolan, los
  eventos sostenidos —un frente de nubosidad— se conservan, porque son la señal que hay que
  aprender.

El repositorio incluye el informe (22 págs.), el documento de continuidad para quien retome el
trabajo (31 págs.), la **derivación matemática completa de XGBoost** —pérdida, aproximación de
Taylor de segundo orden, Similarity Score, ganancia y poda— y el pipeline ejecutado con todas
las salidas guardadas.

`Python` · `XGBoost` · `scikit-learn` · `scikit-optimize` · `SHAP` · `pandas` · `NumPy`

### Dashboard de comparación de modelos de inferencia solar

**[`solar-web`](https://github.com/MatiasLopezING/solar-web)** · LEICI

Front end para comparar en tiempo real las predicciones de distintos modelos de inferencia
climática solar (GHI y K<sub>t</sub>) contra los datos medidos. Gráficos con zoom y
desplazamiento, cambio de modelo y de magnitud en vivo, publicado por GitHub Pages contra una
API expuesta mediante túnel.

`JavaScript` · `Chart.js` · `GitHub Actions` · `Cloudflare Tunnel`

### Minería de datos — análisis de combates de UFC

Trabajo final de la materia: estudio en profundidad de peleas de UFC, con informe extenso
sobre preparación del dato, análisis exploratorio, modelado y discusión de resultados.
*Materia aprobada; el repositorio está en preparación.*

### En curso

**Deep learning** — cursando actualmente. A medida que avance la materia voy a publicar las
implementaciones acá.

---

## Robótica y sistemas embebidos

Es el área que más me está gustando, y donde el diseño se vuelve concreto: un número mal
calculado se nota porque el brazo no levanta el vaso.

### Brazo robótico teleoperado de cinco servomotores

**[`brazo-robotico-taller-proyecto-1`](https://github.com/MatiasLopezING/brazo-robotico-taller-proyecto-1)**
· Taller de Proyecto I (E0306), Departamento de Electrotecnia, FI-UNLP · Grupo N.º 3

Brazo de cuatro grados de libertad más pinza sobre **EDU-CIAA-NXP** (NXP LPC4337, ARM
Cortex-M4), comandado por dos joysticks analógicos, para manipular recipientes a distancia en
entornos con riesgo químico. El caso de uso —trasladar un vaso con líquido sin derramarlo—
impone las dos condiciones que estructuran todo el diseño: el recipiente debe mantenerse
vertical y el movimiento tiene que ser suave.

**El número que define el proyecto.** Con 32,5 cm de alcance y 145 g en la punta, el hombro
necesita **11,96 kg·cm**; el MG996R entrega **9,52 kg·cm** a la tensión de trabajo, y los cinco
servomotores ya estaban comprados. La salida fue interponer un **par de engranajes impresos en
PETG** entre cada servomotor y su articulación: con reducción 3:1 el hombro pasa a exigir
3,99 kg·cm y el diseño cierra con **19 % de margen sobre un factor de seguridad de 2**. Esa
decisión arrastró todo lo demás, incluida la posibilidad de alimentar desde el **riel de +5 V
de una fuente ATX sin ningún convertidor** (4,71 A en régimen, 6,22 A de pico con arranque
escalonado).

Mi parte en el grupo: **dimensionamiento mecánico, transmisión y alimentación**. Los cuadernos
de cálculo —estática en el peor caso, selección de la relación de transmisión, consumo y
dimensionado de fuente, capacitor, cable y fusible— están versionados en el repo.

El proyecto mantiene una **[wiki con siete ADR](https://github.com/MatiasLopezING/brazo-robotico-taller-proyecto-1/wiki)**
(registros de decisión de arquitectura), convenciones de ramas y commits, y un tag por entrega.
Vienen el diseño de PCB y los protocolos de ensayo.

`C` · `EDU-CIAA-NXP / LPC4337` · `sAPI / LPCOpen` · `PWM` · `ADC` · `KiCad` · `Python` (cálculo)

### Sistemas de Tiempo Real

**[`Tiempo-Real`](https://github.com/MatiasLopezING/Tiempo-Real)** · Ingeniería en Computación, 2026

Interrupciones sobre AVR (polling vs. interrupciones, ADC) y planificación con **FreeRTOS**:
tareas concurrentes, sincronización y justificación de las decisiones de diseño. Simulación en
Proteus con los `.hex` y `.elf` de cada caso versionados para poder reproducirlos.

`C` · `AVR` · `FreeRTOS` · `Proteus` · `Atmel Studio`

---

## Hardware, arquitectura y diseño digital

Voy a ser ingeniero en computación: el hardware no es un accesorio del software, es la mitad de
la carrera. Estos repos muestran el trabajo en los niveles de abajo.

### Circuitos Digitales y Microcontroladores

**[`CircuitosDigitalesYMicrocontroladores`](https://github.com/MatiasLopezING/CircuitosDigitalesYMicrocontroladores)**

Trabajos sobre microcontrolador con **drivers escritos desde cero**: `lcd`, `keypad`, `i2c`,
`dht11` (sensor de temperatura y humedad por protocolo de un hilo), `ds3231` (RTC por I²C),
`timer` y `actuadores`, organizados en módulos `.c`/`.h` y gobernados por una **máquina de
estados finitos**. Cada entrega con su proyecto de Proteus para simular el circuito completo.

`C` · `AVR` · `I²C` · `Microchip Studio` · `Proteus`

### Taller de Arquitectura — procesador TDA 1819

**[`TallerDeArquitectura`](https://github.com/MatiasLopezING/TallerDeArquitectura)**

Extensión del procesador didáctico TDA 1819 en **VHDL** para dar soporte a pila: instrucciones
`pushh` / `poph` y un mecanismo para leer y escribir valores de 16 bits en el stack sin
apilar ni desapilar. Validado con una batería de programas en **Assembly** que cubren riesgos
de datos (WAW, store/write-back) y saltos. Incluye además un multiplicador de 4 bits
controlado por FSM, con su testbench.

`VHDL` · `Assembly` · `Active-HDL`

### Introducción al Diseño Lógico

**[`introduccion-diseno-logico`](https://github.com/MatiasLopezING/introduccion-diseno-logico)**

Aritmética de **punto fijo en C** sin `float`, `double` ni `math.h`; máquina de estados de
semáforos y contador Up/Down de 12 bits en **Quartus**, verificados con waveforms. El repo
versiona fuentes y deja fuera enunciados e informes de cátedra a propósito.

`C (C11)` · `Intel Quartus` · `VHDL` · `Verilog`

---

## Software y sistemas

### RecetApp — G2 Innovation

**[`G2_INNOVATION`](https://github.com/MatiasLopezING/G2_INNOVATION)** · Ingeniería de Software

Plataforma web de búsqueda, compra y gestión de medicamentos con tres roles y flujos completos
para cada uno: el **usuario** explora productos cercanos en una vista partida lista + mapa,
arma el carrito y sube la receta; la **farmacia** publica productos, revisa recetas y aprueba
altas de delivery; el **distribuidor** administra pedidos. Las altas de delivery se crean desde
una Cloud Function para no perder la sesión de la farmacia.

`React 19` · `React Router` · `Tailwind` · `MUI` · `Leaflet` · `Firebase (Auth / RTDB / Functions)`

### Sistema de gestión de reparaciones

**[`Reparaciones-EnlacesComunicaciones`](https://github.com/MatiasLopezING/Reparaciones-EnlacesComunicaciones)**

Sistema desplegado para un taller de reparaciones: flujo de estados
pendiente → reparado → retirado, búsqueda, registro de costos y exportación a Excel.
Autenticación con **JWT**, `helmet`, **rate limiting** contra fuerza bruta, CORS configurado,
y **PostgreSQL con SSL** y migraciones automáticas al iniciar.

`Node.js` · `Express` · `PostgreSQL (Neon)` · `JWT` · `PWA`

### CryptoWallet

**[`CryptoWallet-Java`](https://github.com/MatiasLopezING/CryptoWallet-Java)**

Billetera de criptomonedas y moneda fiat en Java: login, balances, transacciones, compra, swap,
cotizaciones en tiempo real contra API y exportación a CSV. Arquitectura por capas con **MVC,
DAO y Singleton**, excepciones propias, manejo de concurrencia y persistencia en SQLite.

`Java` · `Swing` · `SQLite` · `JDBC`

### Redes de Datos II

**[`redes2`](https://github.com/MatiasLopezING/redes2)**

Topología **spine-leaf** con direccionamiento IPv4 e IPv6 montada en CORE. La configuración va
en un script que autodetecta el nodo, no clickeada en la GUI — así la red se levanta igual en
la máquina de cualquiera del grupo.

`CORE` · `Linux` · `Bash` · `IPv4/IPv6` · `Wireshark`

---

## Herramientas

| | |
|---|---|
| **Datos** | Python, pandas, NumPy, scikit-learn, XGBoost, SHAP, Matplotlib, Jupyter |
| **Embebidos** | C, AVR, LPC4337 / ARM Cortex-M4, FreeRTOS, sAPI, LPCOpen, I²C, PWM, ADC |
| **Diseño digital** | VHDL, Verilog, Assembly, Quartus, Active-HDL, Proteus, KiCad |
| **Software** | JavaScript, React, Node.js, Express, Java, PostgreSQL, SQLite, Firebase |
| **Entorno** | Linux (Arch), Git, Bash, LaTeX, GitHub Actions |

---

## Contacto

[LinkedIn](https://www.linkedin.com/in/ingmatiaslopez/) ·
[Repositorios](https://github.com/MatiasLopezING?tab=repositories&sort=updated)
