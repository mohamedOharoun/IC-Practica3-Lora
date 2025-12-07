# IC-Practica3-Lora

## Introducción
Este proyecto implementa un **protocolo de comunicación LoRa personalizado**, diseñado para permitir el envío de mensajes estructurados entre dos nodos que actúan como *Master* y *Slave*. El objetivo principal es crear una arquitectura de intercambio de paquetes fiable, con tipos de mensajes diferenciados, control de errores y confirmaciones.

## Autores
- Wail Ben El Hassane Boudhar
- Mohamed O. Haroun Zarkik

---

## Protocolo de comunicación

El protocolo se basa en **tipos de mensajes codificados en cabeceras**, respetando una estructura fija:

- **MSG_TYPE_DATA (0x01)**: mensajes de datos enviados del Master al Slave.  
- **MSG_TYPE_ECHO (0x02)**: respuesta del Slave confirmando recepción.  
- **MSG_TYPE_ERROR (0x03)**: mensaje generado cuando un paquete es inválido o tiene un tipo desconocido.

Cada paquete enviado contiene:
1. Tipo de mensaje  
2. Longitud  
3. Payload  

Esta estructura permite que los nodos verifiquen que el contenido recibido coincide con lo esperado antes de procesarlo.

---

## Lógica principal del Master

El Master envía periódicamente mensajes de tipo DATA y espera la respuesta ECHO. Si no la recibe, establece una condición de error y reintenta el envío.

Fragmento relevante (primeras ~20 líneas significativas):
```cpp
#define MSG_TYPE_DATA 0x01
#define MSG_TYPE_ECHO 0x02
#define MSG_TYPE_ERROR 0x03

void sendMessage(byte type, const String& payload) {
    LoRa.beginPacket();
    LoRa.write(type);
    LoRa.write(payload.length());
    LoRa.print(payload);
    LoRa.endPacket();
}
```

El Master usa sendMessage() para empaquetar la cabecera y el contenido. Tras enviar, permanece a la espera de un paquete entrante cuyo tipo debe ser MSG_TYPE_ECHO.
```cpp
int packetSize = LoRa.parsePacket();
if (packetSize) {
    byte type = LoRa.read();
    int len = LoRa.read();
    String payload = "";
    while (LoRa.available()) payload += (char)LoRa.read();

    if (type == MSG_TYPE_ECHO) {
        // Confirmación recibida
    } else {
        // Error o mensaje inesperado
    }
}
```

## Lógica principal del Slave

El Slave espera paquetes, analiza su tipo y responde con un ECHO cuando recibe un DATA válido.
```cpp
int packetSize = LoRa.parsePacket();
if (packetSize) {
    byte type = LoRa.read();
    int len = LoRa.read();
    String payload = "";
    while (LoRa.available()) payload += (char)LoRa.read();

    if (type == MSG_TYPE_DATA) {
        sendEcho(payload);
    } else {
        sendError("Tipo no reconocido");
    }
}

```
La función sendEcho() reenvía al Master la carga recibida, permitiendo validar que el mensaje llegó íntegro.
```cpp
void sendError(const String& message) {
    LoRa.beginPacket();
    LoRa.write(MSG_TYPE_ERROR);
    LoRa.write(message.length());
    LoRa.print(message);
    LoRa.endPacket();
}
```

## Pruebas realizadas y trabajo de campo
Para la puesta a prueba del código final, primero se hizo una comprobación de que las dos placas se comunican de forma cercana. Tras ello se decidió poner a prueba el proyecto a aproxidamadamente 1,45km de distancia. La prueba se hizo en el municipio de Santa Lucía de Tirajana en las siguientes ubicaciones:
- En una pequeña montaña de la localidad (27°51'52.6"N 15°27'25.1"W).
- En un punto de la población de Vecindario (27°51'29.6"N 15°26'38.9"W).
Se comparten unas fotos para resaltar el trabajo de campo realizado.

En este vídeo se ve el punto de vista de la placa puesta en la montaña.

https://github.com/user-attachments/assets/dc1c2d84-1f05-4665-afda-0e87eed8818f

El área en rojo se ubicaría de forma aproximada la posición de la otra placa.

<img width="427" height="756" alt="image" src="https://github.com/user-attachments/assets/92399a1d-aafe-4acc-a30e-f4db8e27b77c" />

Últimas fotos que muestran la presencia de una central eólica en las proximidades que imposibilitaban aumentar la distancia
![Untitled-1](https://github.com/user-attachments/assets/4819c3d6-8898-48d0-8485-7ce4ff70c874)

![Untitled](https://github.com/user-attachments/assets/1e9bdf0d-fa86-4c72-bf47-650ffd543e14)



