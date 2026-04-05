# ESP32 BLE Keyboard — HID Attack Demo

 
> Hecho con un **ESP32 DEVKIT V1** y la librería **BleKeyboard**.



## ¿Qué hace esto?

El ESP32 se conecta a una PC por Bluetooth haciéndose pasar por un teclado. Una vez conectado, ejecuta una secuencia automática de teclas que:

1. Abre una terminal CMD con `Win + R`
2. La pone en color verde
3. Ejecuta comandos de diagnóstico de red: `ipconfig /all`, `arp -a`, `netstat -an`
4. Abre Chrome y navega a un video de YouTube

Todo sin tocar la PC. Solo con el ESP32 energizado y emparejado.



##  ¿Cómo funciona?

El ESP32 usa el protocolo **HID (Human Interface Device)** sobre Bluetooth Classic. Cuando se conecta, manda un **HID Descriptor** que básicamente le dice al sistema operativo:

```
"Hola, soy un teclado, aquí están mis keycodes"
```

Windows lo acepta sin instalar drivers. Luego el ESP32 manda paquetes de 8 bytes por cada tecla presionada con el formato:

```
[Modificadores] [Reservado] [Tecla1] [Tecla2] ... [Tecla6]
```

La librería **BleKeyboard** maneja todo eso por nosotros. Nosotros solo decimos `bleKeyboard.print("hola")` y listo.


##  Hardware necesario

- **ESP32 DEVKIT V1** (o cualquier ESP32 clásico con Bluetooth)
- Cable USB para programarlo
- Fuente de energía: puede ser la propia PC, un cargador de celular o un powerbank. **No necesita estar conectado a la PC víctima**, solo necesita corriente.


##  Software y librerías

### Arduino IDE
Descárgalo desde [arduino.cc](https://www.arduino.cc/en/software) si no lo tienes.

### Soporte para ESP32
1. Abre Arduino IDE
2. Ve a **File → Preferences**
3. En **Additional boards manager URLs** pega esto:
```
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```
4. Ve a **Tools → Board → Boards Manager**
5. Busca `esp32 by Espressif Systems`
6. Instala la versión **2.0.17** (importante, la versión 3.x.x da errores de compatibilidad con BleKeyboard)

### Librería BleKeyboard
1. Ve a este repo: [https://github.com/T-vK/ESP32-BLE-Keyboard](https://github.com/T-vK/ESP32-BLE-Keyboard)
2. Click en **Code → Download ZIP**
3. En Arduino IDE ve a **Sketch → Include Library → Add .ZIP Library**
4. Selecciona el ZIP que descargaste
5. Listo 


##  Configurar la placa en Arduino IDE

1. Conecta el ESP32 a tu PC por USB
2. **Tools → Board → ESP32 Arduino → ESP32 Dev Module**
3. **Tools → Port** → selecciona el puerto que aparece (COM3, COM4, etc.)
4. Puedes checar que puerto COM es del esp32 abriendo el admisnitrador de dispositivos y deplegando la opcion de puertos COM.

---

##  Código completo
// BleKeyboard bleKeyboard("NombreQueApareceraEnLaOpcionConectar", "Dev", 100);
 es importante modificar esta linea, y cambiar al nombre que qquieres que aparezca en las opciones de BT de la PC victima

```cpp
#include <BleKeyboard.h>

BleKeyboard bleKeyboard("NombreQueApareceraEnLaOpcionConectar", "Dev", 100);

void setup() {
  Serial.begin(115200);
  bleKeyboard.begin();

  Serial.println("Esperando conexion Bluetooth...");
  while (!bleKeyboard.isConnected()) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nConectado!");

  // Espera larga para que Windows termine de conectar bien
  delay(5000);

  // 1. Abrir CMD con Win+R
  bleKeyboard.press(KEY_LEFT_GUI);
  bleKeyboard.press('r');
  bleKeyboard.releaseAll();
  delay(1500);

  bleKeyboard.print("cmd");
  delay(400);
  bleKeyboard.press(KEY_RETURN);
  bleKeyboard.releaseAll();
  delay(3000);

  // 2. Color verde
  bleKeyboard.print("color 0A");
  delay(300);
  bleKeyboard.press(KEY_RETURN);
  bleKeyboard.releaseAll();
  delay(500);

  // 3. Titulo de ventana
  bleKeyboard.print("title Diagnostico de Red - Isai");
  delay(300);
  bleKeyboard.press(KEY_RETURN);
  bleKeyboard.releaseAll();
  delay(500);

  // 4. ipconfig /all
  bleKeyboard.print("ipconfig /all");
  delay(300);
  bleKeyboard.press(KEY_RETURN);
  bleKeyboard.releaseAll();
  delay(4000);

  // 5. arp -a (equipos en la red local)
  bleKeyboard.print("arp -a");
  delay(300);
  bleKeyboard.press(KEY_RETURN);
  bleKeyboard.releaseAll();
  delay(2000);

  // 6. netstat -an (conexiones activas)
  bleKeyboard.print("netstat -an");
  delay(300);
  bleKeyboard.press(KEY_RETURN);
  bleKeyboard.releaseAll();
  delay(4000);

  // 7. Abrir Chrome con Win+R
  bleKeyboard.press(KEY_LEFT_GUI);
  bleKeyboard.press('r');
  bleKeyboard.releaseAll();
  delay(1500);

  bleKeyboard.print("chrome.exe");
  delay(400);
  bleKeyboard.press(KEY_RETURN);
  bleKeyboard.releaseAll();
  delay(4000);

  // 8. Ctrl+L para ir a la barra de direcciones
  bleKeyboard.press(KEY_LEFT_CTRL);
  bleKeyboard.press('l');
  bleKeyboard.releaseAll();
  delay(800);

  // 9. URL del sitio a donde lo quieres llevar o lo que busque en GOOGLE
  bleKeyboard.print("https://youtu.be/dteTeN4SVHk");
  delay(400);
  bleKeyboard.press(KEY_RETURN);
  bleKeyboard.releaseAll();

  Serial.println("Secuencia completada!");
}

void loop() {}
```

---

##  Subir el código

1. Click en **Upload** (la flecha  en Arduino IDE)
2. Espera a que diga **Done uploading**



##  Emparejar el ESP32 como teclado Bluetooth

1. En tu PC ve a **Configuración → Bluetooth & devices**
2. Activa el Bluetooth
3. Click en **Add device → Bluetooth**
4. Espera y aparece **"NommbreQueLePusiste"**
5. Click en él y listo 


## Ver qué está pasando (Serial Monitor)

1. En Arduino IDE: **Tools → Serial Monitor**
2. Velocidad: **115200 baud**
3. Verás algo así:

```
Esperando conexion Bluetooth...
....
Conectado!
Secuencia completada!
```

---

##  Problemas comunes

| Error | Solución |
|-------|----------|
| Error de compilación con `std::string` | Baja el ESP32 core a la versión **2.0.17** en Boards Manager |
| El link de YouTube sale con "Ñ" | Quita el `?si=...` del final del link, son caracteres que el teclado español no mapea bien |
| La secuencia empieza antes de conectar | Aumenta el `delay(5000)` después de "Conectado!" |
| No aparece en Bluetooth | Abre el Serial Monitor para verificar que el ESP32 está corriendo |


## ¿Necesita estar conectado a la PC objetivo?

**No.** Solo necesita corriente. Puedes energizarlo desde:
 Un cargador de celular normal
 Un powerbank
 Cualquier puerto USB

El Bluetooth funciona de forma inalámbrica. La PC lo detecta igual.




## ⚠️ Aviso legal

Este proyecto fue desarrollado con fines **estrictamente educativos** y probado únicamente en equipos propios en un entorno controlado.

**El autor "Yo" no se hace responsable del uso que terceros hagan de este código o de las técnicas aquí documentadas.** Usar estas técnicas en dispositivos ajenos sin autorización expresa es ilegal y puede tener consecuencias legales serias.

Úsalo para aprender. Úsalo con ética. 