# ESP32 BLE Keyboard - Android

Proyecto para hacer que el ESP32 se conecte al telefono por bluetooth y se haga pasar por un teclado, luego con ayuda de MacroDroid abre el navegador solo y escribe la URL de github donde ya tengo la sesion iniciada con cookies.

---

## Como funciona

El ESP32 manda un HID Descriptor por bluetooth que le dice al telefono "soy un teclado", Android lo acepta sin instalar nada. El problema es que Android no tiene atajos de teclado para abrir apps como Windows, entonces MacroDroid hace esa parte, detecta cuando el ESP32 se conecta y abre el navegador. De ahi el ESP32 escribe la URL solo.

---

## Lo que necesitas

**Hardware**
- ESP32 DEVKIT V1
- Cable USB para programarlo
- Cualquier fuente de energia USB (cargador, powerbank, lo que sea), no necesita estar conectado al telefono

**Apps**
- Arduino IDE en la PC
- MacroDroid en el telefono, esta gratis en Play Store

---

## Instalar soporte para ESP32 en Arduino IDE

Abrir Arduino IDE e ir a File, Preferences. Buscar el campo que dice Additional boards manager URLs y pegar esto:

```
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

Luego ir a Tools, Board, Boards Manager, buscar "esp32 by Espressif Systems" e instalar la version **2.0.17**. No instalar la 3.x.x porque da errores de compatibilidad con la libreria.

---

## Instalar la libreria BleKeyboard

Entrar a este repo y descargar el ZIP:

```
https://github.com/T-vK/ESP32-BLE-Keyboard
```

En Arduino IDE ir a Sketch, Include Library, Add .ZIP Library y seleccionar el ZIP que se descargo.

---

## Configurar la placa

1. Conectar el ESP32 a la PC
2. Tools, Board, ESP32 Arduino, ESP32 Dev Module
3. Tools, Port, seleccionar el puerto que aparece (COM3, COM4, etc.)

---

## Codigo

```cpp
#include <BleKeyboard.h>

BleKeyboard bleKeyboard("ESP32 Teclado Isai", "Isai Dev", 100);

void setup() {
  Serial.begin(115200);
  bleKeyboard.begin();

  Serial.println("Esperando conexion Bluetooth...");
  while (!bleKeyboard.isConnected()) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nConectado!");

  // espera a que MacroDroid abra el navegador
  delay(4000);

  // selecciona la barra de direcciones
  bleKeyboard.press(KEY_LEFT_CTRL);
  bleKeyboard.press('l');
  bleKeyboard.releaseAll();
  delay(800);

  // escribe la url
  bleKeyboard.print("github.com");
  delay(400);
  bleKeyboard.press(KEY_RETURN);
  bleKeyboard.releaseAll();

  Serial.println("listo");
}

void loop() {}
```

---

## Subir el codigo

Click en Upload y esperar a que diga Done uploading.

---

## Configurar MacroDroid

Abrir MacroDroid y crear una macro nueva. En Triggers agregar uno de tipo Bluetooth, Dispositivo conectado y seleccionar "ESP32 Teclado Isai". En Actions agregar Aplicaciones, Abrir aplicacion y elegir el navegador. Guardar.

---

## Emparejar el ESP32

Ir a Configuracion, Bluetooth en el telefono y buscar "ESP32 Teclado Isai". Tocarlo para emparejar.

---

## Ver que esta pasando

Abrir Tools, Serial Monitor en Arduino IDE con velocidad 115200. Deberia salir algo asi:

```
Esperando conexion Bluetooth...
....
Conectado!
listo
```

---

## Errores comunes

Error de compilacion con std::string — bajar el core a la version 2.0.17

MacroDroid no detecta el ESP32 — verificar que este emparejado primero en ajustes de bluetooth

El navegador abre pero no escribe — aumentar el delay de 4000 a algo mas grande

Ctrl+L no funciona — depende del navegador, probar con otro

---

## Nota

Esto se hizo y probo en equipos propios con fines de aprendizaje. No me hago responsable de como lo use alguien mas.