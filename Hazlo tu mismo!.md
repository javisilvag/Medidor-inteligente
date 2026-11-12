# 🌸 Guía DIY: Crea tu propio Medidor Inteligente de Humedad

![ESP32](https://img.shields.io/badge/Hardware-ESP32-red?style=for-the-badge&logo=espressif)
![TFT GC9A01](https://img.shields.io/badge/Display-GC9A01_TFT-blue?style=for-the-badge)
![Open Source](https://img.shields.io/badge/Open_Source-%E2%9D%A4-green?style=for-the-badge)
![License](https://img.shields.io/badge/Licencia-MIT-orange?style=for-the-badge)

En esta guía te enseñamos cómo armar este dispositivo por tu cuenta, compartiéndote nuestro proceso de pruebas, soluciones de circuito y creación de interfaz gráfica para que puedas replicarlo desde tu casa.

---

## 🛠️ 1. Componentes y Conexiones del Hardware

Para que el dispositivo sea compacto y fácil de transportar, eliminamos la placa de pruebas (protoboard) y cambiamos la placa Arduino inicial por una **ESP32**.

### 🛒 Materiales necesarios
* 🧠 **ESP32** (Placa microcontroladora)
* 📺 **GC9A01A** (Pantalla circular TFT)
* 🌾 **HD-38** (Sensor analógico de humedad de suelo)
* 🔌 Cables jumper **Hembra - Hembra** (pantalla)
* 🔌 Cables jumper **Hembra - Macho** (sensor)

### 📌 Tablas de Conexiones (Pinout)

#### Conexión Pantalla TFT GC9A01A ↔ ESP32-S3 SuperMini
| Pin Pantalla TFT | Función / Descripción | Pin ESP32-S3 SuperMini | Tipo de Cable |
|---|---|---|---|
| **VCC** | Alimentación (5V) | **5V** | Hembra - Hembra |
| **GND** | Tierra / Polo negativo | **GND** | Hembra - Hembra |
| **SCL** | Reloj SPI (Clock) | **GPIO 12** | Hembra - Hembra |
| **SDA** | Datos SPI (MOSI) | **GPIO 11** | Hembra - Hembra |
| **DC** | Selección Data / Comando | **GPIO 6** | Hembra - Hembra |
| **CS** | Selección de Chip (Chip Select) | **GPIO 10** | Hembra - Hembra |
| **RST** | Reinicio (Reset) | **GPIO 9** | Hembra - Hembra |

#### Conexión Sensor HD-38 ↔ ESP32-S3 SuperMini
| Pin Módulo HD-38 | Función / Descripción | Pin ESP32-S3 SuperMini | Tipo de Cable |
|---|---|---|---|
| **- (GND)** | Tierra / Polo negativo | **GND** | Hembra - Macho |
| **+ (VCC)** | Alimentación (3.3V) | **3V3** | Hembra - Macho |
| **AO** | Salida analógica (Humedad) | **GPIO AO** | Hembra - Macho |
| **DO** | Salida digital | *(Libre / Sin conectar)* | - |

<pre>
ESP32-S3 SuperMini                  PANTALLA TFT GC9A01A
┌────────────────────┐               ┌────────────────────┐
│                 5V ├──────────────►│ VCC                │
│                GND ├──────┬───────►│ GND                │
│            GPIO 12 ├──────┼───────►│ SCL                │
│            GPIO 11 ├──────┼───────►│ SDA                │
│             GPIO 6 ├──────┼───────►│ DC                 │
│            GPIO 10 ├──────┼───────►│ CS                 │
│             GPIO 9 ├──────┼───────►│ RST                │
│                3V3 ├───┐  │        └────────────────────┘
│            GPIO AO ├──┐│  │
└────────────────────┘  ││  │        SENSOR HD-38
                        ││  │        ┌────────────────────┐
                        ││  └───────►│ - (GND)            │
                        │└──────────►│ + (VCC)            │
                        └───────────►│ AO                 │
                                     │ DO (Libre)         │
                                     └────────────────────┘
</pre>
<img src="../Imagenes%20Evidencias/circuito-compacto.jpg" alt="Circuito compacto sin protoboard" width="350"/>

> [!WARNING]
> **Lección importante sobre la protoboard:**  
> Cuando probamos conectar el sensor a la protoboard, este no funcionaba porque la matriz no hacía buen contacto con los pines del sensor. La solución fue sacarlo de la protoboard y dejar los cables "al aire" (conectados directo). Al hacerlo así, la señal se transmitió bien y el sensor midió correctamente la humedad.

---

## 🎨 2. Creación de la Interfaz Visual (Pixel Art a Código)

Para que la pantalla muestre los estados de la planta de forma empática y visual, creamos dibujos propios que luego convertimos a código para incluir en la pantalla.

### Paso 1: Diseñar las caritas en Pixel Art
Ilustramos las caritas de los cuatro estados según el porcentaje de humedad presente en la tierra:

| Seco (< 30%) | Óptimo (30% - 70%) | Alto (71% - 85%) | Exceso (> 85%) |
|:---:|:---:|:---:|:---:|
| ![Seco](<Imagenes Evidencias/Pixelart triste.gif>) | ![Óptimo](<Imagenes Evidencias/Pixelart feliz.gif>) | ![Alto](<Imagenes Evidencias/Pixelart sorprendido.gif>) | ![Exceso](<Imagenes Evidencias/Pixelart dead.gif>) |
| 😢 Triste | 😊 Sonriente | 😲 Sorprendida | 😵 Ojos en "X" |

### Paso 2: Convertir las imágenes a código C Array
Una vez que tengas tus dibujos listos, no los subas directamente como imágenes pesadas a la placa. Tienes que pasarlos a un formato que la ESP32 pueda leer:

1. Toma la imagen que creaste en Pixel Art.
2. Usa un convertidor en línea como [FileToCArray](https://notisrac.github.io/FileToCArray/) o `image2cpp`.
3. Copia el arreglo de código hexadecimal (`C/C++ array`) generado y pégalo en tu código de Arduino IDE.

### Paso 3: Corrección de formatos de color y tamaño en la pantalla
Al cargar el código para mostrar las imágenes en la pantalla circular, revisa estos detalles:

* 📐 **Ajuste de tamaño:** Adapta las dimensiones del dibujo en el código para que encaje centrado en la pantalla circular.
* 🎨 **Formato de color (RGB vs BGR):** Verifica en el código qué formato de color utiliza la librería de tu pantalla. Si no lo configuras bien, los colores se verán invertidos (por ejemplo, los tonos café podrían verse de color azul). Cambia el formato de color en la librería hasta que los tonos coincidan con tu dibujo original.

---

## 🧪 3. Pruebas de Código y Funcionamiento

1. **Iteración de código:** Al momento de programar, prueba diferentes versiones del código de lectura. En nuestras pruebas nos pasó que algunos códigos hacían encender la pantalla pero el sensor de humedad no respondía; fuimos ajustando el código hasta lograr que la pantalla y el sensor leyeran y mostraran los datos en tiempo real simultáneamente.
2. **Prueba en maceta:** Comprueba que al meter el sensor en la tierra, la pantalla cambie automáticamente entre las caritas en Pixel Art y el porcentaje numérico según los 4 rangos establecidos.

> [!TIP]
> 🎥 **Prueba inicial con Arduino (Prototipo 1):**  
> Esta demostración corresponde al **Prototipo 1**, donde utilizamos una placa Arduino para comprobar la lectura del sensor y verificar que todos los materiales estuvieran en buen estado antes de realizar la migración definitiva a la ESP32-S3 SuperMini.
> 
> [![Ver video en Instagram](Imagenes%20Evidencias/Video%20demostracion.JPG)](https://www.instagram.com/reel/DeSw9m6N9T9/)  
> *(Haz clic en la imagen para abrir el Reel en Instagram ↗️)*

---

## 📦 4. Encapsulado del Dispositivo (Carcasa 3D)

Una vez que el circuito esté listo y funcionando correctamente, puedes guardar y proteger este artefacto de la manera que prefieras:

* 📦 **Opción libre (Cualquier caja):** Puedes poner el circuito dentro de cualquier caja que tengas a mano en tu casa.
* 🌻 **Opción impresión 3D (Recomendada):** Si prefieres algo a medida, puedes diseñar y crear tu propia impresión 3D para meter el artefacto (nosotras, por ejemplo, diseñamos una carcasa con forma de flor).

[<img src="../Imagenes%20Evidencias/Evidencia%205.jpg" alt="Render de la carcasa con forma de flor" width="350"/>](https://github.com/user-attachments/assets/d262300f-91b0-463c-8c8f-ec3e6bab44b7)

> 💡 **Requisito clave:** Lo único importante es que tu caja o carcasa deje visible la pantalla circular y permita que la sonda del sensor quede libre para clavarse en la tierra.
