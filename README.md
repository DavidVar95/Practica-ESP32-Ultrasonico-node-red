# Practica ESP32 con Sensor ultrasonico y node-red
Este repositorio muestra como podemos mandasr datos a un servidor y poder graficarlos
## Introducción

### Descripción

En esta practica utilizamos la ```Esp32``` con un un sensor ultrasonico y en el programa mandamos los datos que obtiene el ESP32 y los mandamos a un servidor publico y con node red podemos visualizar los datos y graficar la informacion

## Material Necesario

Para realizar esta practica necesitas lo siguiente

- [WOKWI](https://https://wokwi.com/)
- Tarjeta ESP 32
- Sensor ultrasonico
- node-red



## Instrucciones


Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://https://wokwi.com/).


### Instrucciones de preparación de entorno 

1. Abrir la terminal de programación y colocar la siguente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="DavidVargas";
String password_mqtt="1234";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;
const int Trigger = 4;   //Pin digital 2 para el Trigger del sensor
const int Echo = 15;   //Pin digital 3 para el Echo del sensor

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
   pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {


delay(1000);
 long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    //doc["TEMPERATURA"] = String(data.temperature, 1);
    //doc["HUMEDAD"] = String(data.humidity, 1);
   doc["DISTANCIA"] =d;

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("Vargas95", output.c_str());
  }
}
```
2. Realizar la conecion del circuito

![](https://github.com/DavidVar95/Practica-ESP32-Ultrasonico-node-red/blob/main/Captura%20de%20pantalla%202023-06-17%2013.21.02.png?raw=true)

3. Se realiza el diagrama de bloques en node red

![](https://github.com/DavidVar95/Practica-ESP32-Ultrasonico-node-red/blob/main/Captura%20de%20pantalla%202023-06-17%2013.24.01.png?raw=true)

4. colocamos un bloque mqttin y lo vinculamos con el programa en WOKWI

![](https://github.com/DavidVar95/Practica-con-DHT-22-Y-node-red/blob/main/Captura%20de%20pantalla%202023-06-16%2022.01.30.png?raw=truee)

5. Configurar el bloque con el puerto mqtt con el ip 44.195.202.69 como se muestra en la imagen

![](https://github.com/DavidVar95/Practica-con-DHT-22-Y-node-red/blob/main/Captura%20de%20pantalla%202023-06-16%2022.05.45.png?raw=true)

6. Colocar el bloque json y configurarlo como se muestra en la imagen

![](https://github.com/DavidVar95/Practica-con-DHT-22-Y-node-red/blob/main/Captura%20de%20pantalla%202023-06-16%2022.07.58.png?raw=true)

7. Colocamos un bloque function y lo configuramos con el siguente codigo

```
msg.payload = msg.payload.DISTANCIA;
msg.topic = "DISTANCIA";
return msg;

```
![](https://github.com/DavidVar95/Practica-ESP32-Ultrasonico-node-red/blob/main/Captura%20de%20pantalla%202023-06-17%2013.28.50.png?raw=true)


# Resultados



En el node-red podememos ver los datos que esta tomando el ESP32


![](https://github.com/DavidVar95/Practica-ESP32-Ultrasonico-node-red/blob/main/Captura%20de%20pantalla%202023-06-17%2013.30.44.png?raw=true)

![](https://github.com/DavidVar95/Practica-ESP32-Ultrasonico-node-red/blob/main/Captura%20de%20pantalla%202023-06-17%2013.30.56.png?raw=truee)

![](https://github.com/DavidVar95/Practica-ESP32-Ultrasonico-node-red/blob/main/Captura%20de%20pantalla%202023-06-17%2013.31.04.png?raw=true)

# Créditos

Desarrollado por Ing. David Vargas Gonzalez

