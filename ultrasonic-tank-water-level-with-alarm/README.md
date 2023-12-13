# ultrasonic-tank-water-level-with-alarm
> ## Code Files

#### distance-measure-ultrasonic.ino
```
#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <ESPAsyncWebSrv.h>
#include <WebSocketsServer.h>
#include <Ticker.h>

// WiFi credentials
const char* WIFI_SSID = "WiFi";
const char* WIFI_Password = "akash@wifi";

// ESP8266
const int trigPin = D0;
const int echoPin = D1;
const int relay = D4;
// ESP01
// const int trigPin = 0;
// const int echoPin = 2;

// Timer for sensor readings
Ticker timer;

char webpage[] PROGMEM = R"=====(
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ESP Control</title>
</head>
<style>
    main {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        gap: 1em;
    }

    h1 {
        text-align: center;
    }

    #tank {
        width: 350px;
        height: 400px;
        border: 5px solid black;
        border-radius: 10px;
        display: flex;
        align-items: end;
        padding: 0 4px;
    }

    #tank div {
        width: 100%;
        background: blue;
    }
</style>

<body>
    <main>
        <div>
            <h1>My Tank for esp8266</h1>
        </div>
        <div id="tank">
            <div id="water"></div>
        </div>
        <h2 id="percentage">%</h2>
        <h2 id="distance"></h2>

    </main>
    <script>
        var tankHeight = 100; //cm
        var tankMargin = 33; //cm (+)
        
        var connection = new WebSocket('ws://' + location.hostname + ':81/');

        connection.onmessage = function (event) {
            var received_distane = event.data
            distance.innerHTML = received_distane + 'cm'
            water_percent = (100 - (((received_distane - tankMargin) / tankHeight) * 100)).toFixed(2)
            percentage.textContent = water_percent + '%'
            water.style.height = water_percent + '%'
            if (water_percent < 0) {
                water.style.height = `${100 - water_percent}%`
                water.style.backgroundColor = 'red'
            }else{
                water.style.backgroundColor = 'blue'
            }
        }
    </script>
</body>

</html>
)=====";



AsyncWebServer server(80); // server port 80
WebSocketsServer websockets(81);


void notFound(AsyncWebServerRequest* request) {
  request->send(404, "text/plain", "Page Not Found");
}

void webSocketEvent(uint8_t num, WStype_t type, uint8_t* payload, size_t length) {
  switch (type) {
    case WStype_DISCONNECTED:
      Serial.printf("[%u] Disconnected!
", num);
      break;
    case WStype_CONNECTED: {
      IPAddress ip = websockets.remoteIP(num);
      Serial.printf("[%u] Connected from %d.%d.%d.%d
", num, ip[0], ip[1], ip[2], ip[3]);
    }
    break;
    case WStype_TEXT:
      Serial.printf("[%u] Received Text: %s
", num, payload);
      // No need to convert payload to a String unless you have specific requirements.
      break;
  }
}

void setup() {
  // Serial.begin(115200);
  Serial.begin(9600);
   Serial.println("Incitization....");

  // Initialize pins
  pinMode(trigPin, OUTPUT);
  pinMode(relay, OUTPUT);
  pinMode(echoPin, INPUT);

  pinMode(LED_BUILTIN, OUTPUT);

  // Connect to WiFi
  WiFi.begin(WIFI_SSID, WIFI_Password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    digitalWrite(LED_BUILTIN, LOW);
  }
  digitalWrite(LED_BUILTIN, HIGH);
  Serial.println("");
  Serial.print("Connected to ");
  Serial.println(WIFI_SSID);
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

  server.on("/", HTTP_GET, [](AsyncWebServerRequest* request) {
    request->send_P(200, "text/html", webpage);
  });
  server.onNotFound(notFound);
  server.begin();

  // Initialize WebSocket server
  websockets.begin();
  websockets.onEvent(webSocketEvent);

  // Set up the timer for sensor readings
  timer.attach(2.0, send_sensor); // Adjust the interval as needed
}

void loop() {
  websockets.loop();
}

void send_sensor() {
  // Trigger and measure distance
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  long duration = pulseIn(echoPin, HIGH);
  int distance_cm = (duration * 0.034) / 2;

  if (distance_cm > 10){
    digitalWrite(relay, HIGH);
  }else{
    digitalWrite(relay, LOW);

  }

  // Broadcast the distance reading to connected clients
  char sensorData[10]; // Create a character array to hold sensor data
  snprintf(sensorData, sizeof(sensorData), "%d", distance_cm);
  Serial.println(distance_cm);

  websockets.broadcastTXT(sensorData);
}
```

<br>

