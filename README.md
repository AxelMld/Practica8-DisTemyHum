# **PRÁCTICA 8 ENVIO DE DATOS A UNA PAGINA WEB**

## **INICIO DE NODEJS** ##

1. Se abre el programa "CMD" en modo administrador.
2. Una vez abierto se ponen el siguiente comando.

                    2. Para arrancar nodejs se usa el comando **node- red** 

Aparecera lo siguiente ya inicio el nodejs, se deja abierto para su uso 

![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/arranque%20nodejs.png?raw=true)

3. Nos Dirigimos al navegador buscamos la pagina **localhost:1880**

Aparecera la siguiente imagen lugar de trabajo de nodejs


![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/nodejs%20trabajo.png?raw=true)

----------------------------------------------------------------

## **REALIZACION Y CONFIGURACION DE ESTRUCTURA EN NODEJS** ##

1. Se realiza la siguiente estructura 

![](https://github.com/AxelMld/Practica8-DisTemyHum/blob/main/Estructura%20nodejs.png?raw=true)

### Configuracion y herramientas usadas 

* Para DAIyM/Axel/P8 se utilizo  **mqtt in** "nombre utilizado en codigo"
* Para json se utilizo        **json**
* Se utilizo  tres   **function** "nombrado cada uno con su respectivo nombre para su medicion"
* Se utilizo tres  **gauge** "nombrado cada uno con su respectivo nombre para su medicion"
* Para Graficar se utilizo    **chart** aqui se grafican los tres datos Distancia, Temeperatura, Humedad.


--------------------------------------------------------------

## CONFIGURACION ESTRUCTURA NODEJS

* Para la configuracion de mqtt in se utilizo la siguiente 
![](https://github.com/AxelMld/Practica8-DisTemyHum/blob/main/configuracion%20mqtt.png?raw=true)

* Para json solo se escogio **Always convert to JavaScrip Object**

* CODIGO de las funciones "function" 

**Distancia**

```
msg.payload = msg.payload.Distancia;
msg.topic = "Distancia";
return msg;

```

**Temperatura**

```
msg.payload = msg.payload.Temperatura;
msg.topic = "Temperatura";
return msg;

```

**Humedad**

```
msg.payload = msg.payload.Humedad;
msg.topic = "Humedad";
return msg;

```
# CONFIGURACIONES TABS & LINKS DE GAUGE Y CHART

* En el apartado dashboard se agrega nueva tabla con las siguientes grupos como se observa en la imagen.

![](https://github.com/AxelMld/Practica8-DisTemyHum/blob/main/conf%20dashboard.png?raw=true)


* Distancia 

![](https://github.com/AxelMld/Practica8-DisTemyHum/blob/main/Distancia.png?raw=true)

* Temperatura 

![](https://github.com/AxelMld/Practica8-DisTemyHum/blob/main/Temperatura.png?raw=true)

* Distancia 

![](https://github.com/AxelMld/Practica8-DisTemyHum/blob/main/Humedad.png?raw=true)

---------------------------------------------------------------

# **CONEXION Y CODIGO EN WOKWI**

## Para la simulación nos dirigimos a la página  

woki : https://wokwi.com

#### Dashboard de página a utilizar 
![](https://github.com/AxelMld/Practica-3-Sensor-Ultrasonico-/blob/main/dash.png?raw=true)


## Para está práctica se utilizó 

* Módulo ESP32 
* HC-SR04 Ultrasonic Distance Sensor
* DHT22


## Pasos a Seguir 

1. Seleccionar el módulo ESP32.
2. Escoger los dispositivos a utilizar anteriormente mensionados. 
3. Realizar las conexiones como se muestra en el apartado "conexión".
4. Agregar las librerias mensiadas en el apartado Librerías 




## **Conexión**

![](https://github.com/AxelMld/Practica8-DisTemyHum/blob/main/conexion.png?raw=true)


## **Librerías**

1. ArduinoJson
2. PubSubClient
3. DHT sensor library for ESPx

## Código utilizado 


```

#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#include "DHTesp.h"

#define BUILTIN_LED 2
const int Trigger = 13;   //Pin digital 2 para el Trigger del sensor
const int Echo = 12;   //Pin digital 3 para el Echo del sensor



//pines y seccion del sensor DHT22
const int DHT_PIN = 15;

DHTesp dhtSensor;


// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="AXELM";
String password_mqtt="1234";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

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

  Serial.begin(9600);//iniciailzamos la comunicación
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
 
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0


}

void loop() {

long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

 TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  Serial.println("Temp: " + String(data.temperature, 1) + "°C");
  Serial.println("Humidity: " + String(data.humidity, 1) + "%");
  Serial.println("---");

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
  
  Serial.print("Distancia: ");
  Serial.print(d);      //Enviamos serialmente el valor de la distancia
  Serial.print("cm");
  Serial.println();
  delay(1000);          //Hacemos una pausa de 100ms

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
    doc["Distancia"] = String(d);
    doc["Temperatura"] = String(data.temperature, 1);
    doc["Humedad"] = String(data.humidity, 1);
    
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("DAIyM/AxelM/P8", output.c_str());
  }
}

```


# **RESULTADO**

## 1. Resultado de la pagina WOKWI 

![](https://github.com/AxelMld/Practica8-DisTemyHum/blob/main/Resultado%20de%20wokwi.png?raw=true)

## 2. Primer  resultado del dashboard 
 
![](https://github.com/AxelMld/Practica8-DisTemyHum/blob/main/resultado%20red.png?raw=true)

## 3. Mas resultados

![](https://github.com/AxelMld/Practica8-DisTemyHum/blob/main/resultado%202.png?raw=true)

![](https://github.com/AxelMld/Practica8-DisTemyHum/blob/main/resultado%202.2.png?raw=true)


**Notas:** Tener cuidado con el nombramiento de la red, asi como nombramiento del client-publish




# Creditos 

**Axel Miranda.**
