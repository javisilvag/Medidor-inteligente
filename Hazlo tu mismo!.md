# 🌸 Guía DIY: Crea tu propio Medidor Inteligente de Humedad

![ESP32](https://img.shields.io/badge/Hardware-ESP32-red?style=for-the-badge&logo=espressif)
![TFT GC9A01](https://img.shields.io/badge/Display-GC9A01_TFT-blue?style=for-the-badge)
![Open Source](https://img.shields.io/badge/Open_Source-%E2%9D%A4-green?style=for-the-badge)
![License](https://img.shields.io/badge/Licencia-MIT-orange?style=for-the-badge)

En esta guía te enseñamos cómo armar este dispositivo por tu cuenta, compartiéndote nuestro proceso de pruebas, soluciones de circuito y creación de interfaz gráfica para que puedas replicarlo desde tu casa.

---

##  1. Componentes y Conexiones del Hardware

Para que el dispositivo sea compacto y fácil de transportar, eliminamos la placa de pruebas (protoboard) y cambiamos la placa Arduino inicial por una **ESP32-S3 SuperMini**.

###  Materiales necesarios
-  **ESP32-S3 SuperMini** (Placa microcontroladora)
-  **GC9A01A** (Pantalla circular TFT de 240×240 px)
-  **HD-38** (Sensor analógico de humedad de suelo)
-  Cables jumper **Hembra - Hembra** (pantalla)
-  Cables jumper **Hembra - Macho** (sensor)

###  Tablas de Conexiones (Pinout)

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

<img src="Imagenes%20Evidencias/Evidencia%202.jpeg" alt="Circuito compacto sin protoboard" width="350"/>

> [!WARNING]
> **Lección importante sobre la protoboard:**  
> Cuando probamos conectar el sensor a la protoboard, este no funcionaba porque la matriz no hacía buen contacto con los pines del sensor. La solución fue sacarlo de la protoboard y dejar los cables "al aire" (conectados directo). Al hacerlo así, la señal se transmitió bien y el sensor midió correctamente la humedad.

---

##  2. Creación de la Interfaz Visual (Pixel Art a Código)

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

### Paso 3: Ajuste de tamaño (100×100 a 240×240 px), memoria y color
Al cargar las ilustraciones en la pantalla circular, tuvimos que realizar tres ajustes técnicos fundamentales:

-  **De 100×100 a 240×240 píxeles:** Nosotras diseñamos originalmente las caritas en **100 × 100 px**, pero la pantalla circular GC9A01A tiene una resolución nativa de **240 × 240 px**. Antes de convertir los dibujos a código, debes redimensionarlos a **240 × 240 px** para que cubran toda la pantalla sin verse pequeños ni descentrados.
-  **Ampliar la memoria en Arduino IDE (*Partition Scheme*):** Al escalar las imágenes de 100×100 a **240×240 px**, los arreglos de código (`C/C++ array`) ocupan mucha más memoria y superan el límite que Arduino IDE asigna por defecto al programa (arrojando el error *"Sketch too big"*). Para que la ESP32-S3 pueda almacenar toda la información:
  1. En Arduino IDE, ve al menú superior **Herramientas** (*Tools*).
  2. Busca la opción **Partition Scheme** (*Esquema de partición*).
  3. Cámbiala de la opción por defecto a **Huge APP (3MB No OTA / 1MB SPIFFS)** *(o `No OTA (2MB APP / 2MB SPIFFS)`)*.
  4. Vuelve a compilar y subir el código: con esto amplías el espacio destinado al programa y la placa soportará todas las caritas sin problemas.
-  **Formato de color (RGB vs BGR):** Verifica en el código qué formato de color utiliza la librería de tu pantalla. Si no lo configuras bien, los colores se verán invertidos (por ejemplo, los tonos café podrían verse de color azul). Cambia el formato de color en la librería hasta que los tonos coincidan con tu dibujo original.

---

##  3. Pruebas de Código y Funcionamiento

1. **Iteración de código:** Al momento de programar, prueba diferentes versiones del código de lectura. En nuestras pruebas nos pasó que algunos códigos hacían encender la pantalla pero el sensor de humedad no respondía; fuimos ajustando el código hasta lograr que la pantalla y el sensor leyeran y mostraran los datos en tiempo real simultáneamente.
2. **Prueba en maceta:** Comprueba que al meter el sensor en la tierra, la pantalla cambie automáticamente entre las caritas en Pixel Art y el porcentaje numérico según los 4 rangos establecidos.

> [!TIP]
>  **Prueba inicial con Arduino (Prototipo 1):**  
> Esta demostración corresponde al **Prototipo 1**, donde utilizamos una placa Arduino para comprobar la lectura del sensor y verificar que todos los materiales estuvieran en buen estado antes de realizar la migración definitiva a la ESP32-S3 SuperMini.
> 
> [![Ver video en Instagram](Imagenes%20Evidencias/Video%20demostracion.JPG)](https://www.instagram.com/reel/DeSw9m6N9T9/)  
> *(Haz clic en la imagen para abrir el Reel en Instagram ↗️)*

---

##  4. Encapsulado del Dispositivo (Carcasa 3D)

Una vez que el circuito esté listo y funcionando correctamente, puedes guardar y proteger este artefacto de la manera que prefieras:

-  **Opción libre (Cualquier caja):** Puedes poner el circuito dentro de cualquier caja que tengas a mano en tu casa.
-  **Opción impresión 3D (Recomendada):** Si prefieres algo a medida, puedes diseñar y crear tu propia impresión 3D para meter el artefacto (nosotras, por ejemplo, diseñamos una carcasa con forma de flor).

[<img src="Imagenes%20Evidencias/Evidencia%205.jpg" alt="Render de la carcasa con forma de flor" width="350"/>](https://github.com/user-attachments/assets/d262300f-91b0-463c-8c8f-ec3e6bab44b7)

>  **Requisito clave:** Lo único importante es que tu caja o carcasa deje visible la pantalla circular y permita que la sonda del sensor quede libre para clavarse en la tierra.

##  5. Código Fuente (Arduino Uno y ESP32-S3)

Aquí tienes los dos códigos que desarrollamos durante el proyecto: primero la versión de diagnóstico en **Arduino Uno** y luego el código definitivo en **ESP32-S3 SuperMini** con las animaciones en Pixel Art.

<details>
<summary><b>🧪 Ver Código 1: Prototipo 1 en Arduino Uno (Calibración, textos y barra de nivel)</b></summary>
<br>

En esta primera etapa conectamos el sensor **HD-38** y la pantalla circular **GC9A01A** a una placa **Arduino Uno**. Como la memoria del Arduino Uno no alcanza para almacenar imágenes pesadas en alta resolución, diseñamos una interfaz inicial basada en porcentajes, textos de estado con colores (`BAJO`, `OPTIMO`, `ALTO`, `EXCESO`) y una barra de progreso dinámica, calibrando el sensor entre `1023` (seco) y `240` (saturado en agua).

> **Librerías necesarias en Arduino IDE:** `Adafruit GFX Library` y `Adafruit GC9A01A`.

<pre>
#include <SPI.h>
#include <Adafruit_GFX.h>
#include <Adafruit_GC9A01A.h>

// Definición de pines para la pantalla TFT en Arduino Uno
#define TFT_RST  8
#define TFT_DC   9
#define TFT_CS   10

// Pin del sensor de humedad analógico (HD-38)
#define PIN_SENSOR A0 

Adafruit_GC9A01A tft(TFT_CS, TFT_DC, TFT_RST);

unsigned long ultimoTiempo = 0;
const long intervalo = 1000; // Lee el sensor cada 1 segundo

// =========================================================================
// VALORES DE CALIBRACIÓN REALES DE TU SENSOR
// =========================================================================
const int VALOR_SECO   = 1023; // 1023 = 0% Humedad (Aire / Seco)
const int VALOR_MOJADO = 240;  // 240  = 100% Humedad (Saturado / Mojado)

void setup() {
  Serial.begin(9600);

  // Inicializar pantalla
  tft.begin();
  tft.setRotation(0);
  tft.fillScreen(GC9A01A_BLACK);

  // Dibujar elementos fijos
  tft.drawCircle(120, 120, 118, GC9A01A_WHITE);
  tft.drawCircle(120, 120, 115, GC9A01A_DARKGREY);
  
  tft.setTextColor(GC9A01A_WHITE);
  tft.setTextSize(2);
  tft.setCursor(65, 45);
  tft.println("HUMEDAD");
}

void loop() {
  if (millis() - ultimoTiempo >= intervalo) {
    ultimoTiempo = millis();

    int lecturaRaw = analogRead(PIN_SENSOR);
    
    // Mapeo preciso con tus valores reales
    int porcentaje = map(lecturaRaw, VALOR_SECO, VALOR_MOJADO, 0, 100);
    porcentaje = constrain(porcentaje, 0, 100);

    // Muestra por Monitor Serie para verificar
    Serial.print("Lectura A0: ");
    Serial.print(lecturaRaw);
    Serial.print(" -> ");
    Serial.print(porcentaje);
    Serial.println("%");

    actualizarPantalla(porcentaje);
  }
}

void actualizarPantalla(int humedad) {
  uint16_t colorEstado;
  String textoEstado = "";

  // Rangos de humedad para tus plantas
  if (humedad < 25) {
    textoEstado = "BAJO (SECO)";
    colorEstado = GC9A01A_RED;         // < 25%: Necesita riego urgente
  } else if (humedad <= 65) {
    textoEstado = "OPTIMO (OK)";
    colorEstado = GC9A01A_GREEN;       // 25% - 65%: Humedad ideal
  } else if (humedad <= 85) {
    textoEstado = "ALTO (MOJADO)";
    colorEstado = GC9A01A_BLUE;        // 66% - 85%: Recién regada
  } else {
    textoEstado = "EXCESO AGUA!";
    colorEstado = GC9A01A_MAGENTA;     // > 85%: Posible encharcamiento
  }

  // 1. Limpiar áreas dinámicas para evitar parpadeos
  tft.fillRect(40, 80, 160, 40, GC9A01A_BLACK);  
  tft.fillRect(15, 130, 210, 30, GC9A01A_BLACK); 
  tft.fillRect(40, 175, 160, 20, GC9A01A_BLACK); 

  // 2. Dibujar Porcentaje
  tft.setTextColor(GC9A01A_WHITE);
  tft.setTextSize(4);
  
  if (humedad < 10) tft.setCursor(100, 85);
  else if (humedad < 100) tft.setCursor(85, 85);
  else tft.setCursor(70, 85);
  
  tft.print(humedad);
  tft.setTextSize(2);
  tft.print("%");

  // 3. Dibujar Texto de Estado
  tft.setTextColor(colorEstado);
  tft.setTextSize(2);
  int offsetTexto = (240 - (textoEstado.length() * 12)) / 2;
  tft.setCursor(offsetTexto, 135);
  tft.print(textoEstado);

  // 4. Dibujar Barra de Nivel
  int anchoBarra = map(humedad, 0, 100, 0, 150);
  tft.drawRect(45, 175, 150, 15, GC9A01A_WHITE);
  tft.fillRect(45, 175, anchoBarra, 15, colorEstado);
}
</pre>

###  Código ESP32
Este es nuestro código definitivo para la **ESP32-S3 SuperMini**. En lugar de imágenes estáticas, diseñamos animaciones de **3 fotogramas (*frames*)** por cada estado emocional (`seco`, `optimo`, `alto` y `exceso`) que se reproducen cada `200 ms` mediante renderizado ultra rápido por SPI (`writePixels`).

> [!NOTE]
> **Detalles técnicos clave en este código:**
> - **Memoria:** Requiere activar el esquema de partición **Huge APP** en Arduino IDE para almacenar los 12 fotogramas en `PROGMEM` (240×240 px cada uno).
> - **Corrección de color:** Incluye `tft.invertDisplay(true)` para calibrar los colores reales de la pantalla circular GC9A01A.
> - **Pinout del sensor:** Usa `GPIO 1` como lectura analógica y `GPIO 4` configurado en `LOW` como tierra (`GND`) dedicada para el sensor HD-38.

<details>
<summary><b>🌸 Ver Código ESP32-S3 SuperMini (.ino)</b></summary>
<br>

<pre>
#include <SPI.h>
#include <Adafruit_GFX.h>
#include <Adafruit_GC9A01A.h>

// Definición de pines Pantalla TFT GC9A01A <-> ESP32-S3 SuperMini
#define TFT_SCL  12
#define TFT_SDA  11
#define TFT_DC    6
#define TFT_CS   10
#define TFT_RST   9

// Pines Sensor de Humedad HD-38
#define SENSOR_PIN 1  // Lectura analógica (AO)
#define SENSOR_GND 4  // Tierra virtual dedicada (GND)

Adafruit_GC9A01A tft = Adafruit_GC9A01A(TFT_CS, TFT_DC, TFT_RST);

// =========================================================================
// ARREGLOS DE IMÁGENES PIXEL ART (12 frames de 240x240 px en PROGMEM)
// Nota: Se omiten los valores hexadecimales por extensión; insertar aquí
// los arreglos generados con FileToCArray / image2cpp.
// =========================================================================

// --- IMÁGENES PARA ESTADO "SECO" (3 frames) ---
static const uint16_t seco1[] PROGMEM = { /* Agregar código imagen */ };
static const uint16_t seco2[] PROGMEM = { /* Agregar código imagen */ };
static const uint16_t seco3[] PROGMEM = { /* Agregar código imagen */ };
const uint16_t* animSeco[3] = {seco1, seco2, seco3};

// --- IMÁGENES PARA ESTADO "OPTIMO" (3 frames) ---
const uint16_t optimo1[] PROGMEM = { /* Agregar código imagen */ };
const uint16_t optimo2[] PROGMEM = { /* Agregar código imagen */ };
const uint16_t optimo3[] PROGMEM = { /* Agregar código imagen */ };
const uint16_t* animOptimo[3] = {optimo1, optimo2, optimo3};

// --- IMÁGENES PARA ESTADO "ALTO" (3 frames) ---
const uint16_t alto1[] PROGMEM = { /* Agregar código imagen */ };
const uint16_t alto2[] PROGMEM = { /* Agregar código imagen */ };
const uint16_t alto3[] PROGMEM = { /* Agregar código imagen */ };
const uint16_t* animAlto[3] = {alto1, alto2, alto3};

// --- IMÁGENES PARA ESTADO "EXCESO" (3 frames) ---
const uint16_t exceso1[] PROGMEM = { /* Agregar código imagen */ };
const uint16_t exceso2[] PROGMEM = { /* Agregar código imagen */ };
const uint16_t exceso3[] PROGMEM = { /* Agregar código imagen */ };
const uint16_t* animExceso[3] = {exceso1, exceso2, exceso3};

int frameActual = 0;
unsigned long ultimoRefresco = 0;
const int intervaloFrame = 200; // 200ms = velocidad pausada de animación

void dibujaImagenUltraRapido(const uint16_t* bitmap) {
  tft.startWrite();
  tft.setAddrWindow(0, 0, 240, 240);
  tft.writePixels((uint16_t*)bitmap, 57600);
  tft.endWrite();
}

void setup() {
  pinMode(SENSOR_GND, OUTPUT);
  digitalWrite(SENSOR_GND, LOW); 

  Serial.begin(115200);

  // Reinicio manual de la pantalla TFT
  pinMode(TFT_RST, OUTPUT);
  digitalWrite(TFT_RST, HIGH);
  delay(10);
  digitalWrite(TFT_RST, LOW);
  delay(20);
  digitalWrite(TFT_RST, HIGH);
  delay(50);

  // Inicializar bus SPI a 80 MHz
  SPI.begin(TFT_SCL, -1, TFT_SDA, TFT_CS);
  tft.begin(80000000); 
  tft.setRotation(2); 
  tft.invertDisplay(true); // Corrección de inversión de color
}

void loop() {
  unsigned long tiempoActual = millis();

  if (tiempoActual - ultimoRefresco >= intervaloFrame) {
    ultimoRefresco = tiempoActual;

    int lecturaRaw = analogRead(SENSOR_PIN);
    // 1400 permite que al sumergirse en agua sobrepase el 75% y active "EXCESO"
    int porcentaje = map(lecturaRaw, 4095, 1400, 0, 100);
    porcentaje = constrain(porcentaje, 0, 100);

    const uint16_t* ptrImagen = NULL;

    // Selección de estado y fotograma actual
    if (porcentaje < 20) { 
      if (frameActual == 0) ptrImagen = seco1;
      else if (frameActual == 1) ptrImagen = seco2;
      else ptrImagen = seco3;
    } 
    else if (porcentaje < 50) { 
      if (frameActual == 0) ptrImagen = optimo1;
      else if (frameActual == 1) ptrImagen = optimo2;
      else ptrImagen = optimo3;
    } 
    else if (porcentaje < 75) { 
      if (frameActual == 0) ptrImagen = alto1;
      else if (frameActual == 1) ptrImagen = alto2;
      else ptrImagen = alto3;
    } 
    else { 
      if (frameActual == 0) ptrImagen = exceso1;
      else if (frameActual == 1) ptrImagen = exceso2;
      else ptrImagen = exceso3;
    }

    if (ptrImagen != NULL) {
      dibujaImagenUltraRapido(ptrImagen);
    }

    frameActual++;
    if (frameActual > 2) {
      frameActual = 0;
    }
  }

  yield();
}
</pre>

---

### 🖼️ Recursos Gráficos: Fotogramas (Frames) para el Código ESP32

Si quieres replicar las animaciones en tu propia placa, descarga los **3 fotogramas individuales** de cada estado emocional desde sus respectivas carpetas y conviértelos a arreglos hexadecimales (`C/C++ array`) para pegarlos en el código:

| Estado de la Planta | Rango de Humedad | Arreglos en el Código | Acceso Directo a los Archivos |
| :--- | :---: | :--- | :--- |
| 😢 **Seco** | `< 20%` | `seco1`, `seco2`, `seco3` | [📂 Abrir carpeta **Triste**](Imagenes%20Evidencias/Imagenes%20Gif/Triste) |
| 😊 **Óptimo** | `20% - 49%` | `optimo1`, `optimo2`, `optimo3` | [📂 Abrir carpeta **Feliz**](Imagenes%20Evidencias/Imagenes%20Gif/Feliz) |
| 😲 **Alto** | `50% - 74%` | `alto1`, `alto2`, `alto3` | [📂 Abrir carpeta **Sorprendido**](Imagenes%20Evidencias/Imagenes%20Gif/Sorprendido) |
| 😵 **Exceso** | `≥ 75%` | `exceso1`, `exceso2`, `exceso3` | [📂 Abrir carpeta **Dead**](Imagenes%20Evidencias/Imagenes%20Gif/Dead) |
