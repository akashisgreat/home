# rfid-nfc-grocery-list-maker
> ## Images

####  RC522-RFID-Reader-Module.jpg
![RC522-RFID-Reader-Module.jpg](./pic/RC522-RFID-Reader-Module.jpg 'RC522-RFID-Reader-Module.jpg')
####  esp-with-rfid-connection.png
![esp-with-rfid-connection.png](./pic/esp-with-rfid-connection.png 'esp-with-rfid-connection.png')
<br>

> ## Budgets

####  price.csv
| ITEMS             | PRICE | WHAT_I_PAID |
| ----------------- | ----- | ----------- |
| Nodemcu           | 300   | 289         |
| RC522 RFID Reader | 200   | 204         |
| Buzzer            | 20    | 0           |
| Jumper wire * 10  | 50    | 12.5        |
| RFID Tag * 1      | 20    | 0           |
| SUM               | 590   | 505.5       |


<br>

> ## Code Files

#### rfid-list-maker-with-telegram-bot.ino
```
/* Pin Connections 
--------------------
ESP | RFID READER
D1 = rst
D2 = sda
D5 = sck
D6 = miso
D7 = mosi
D3 = bizzer/led
3v = 3v/VIN
G = G
------------ */

#include <ESP8266WiFi.h>
#include <WiFiClientSecure.h>
#include <SPI.h>
/* first Download these following */
#include <MFRC522.h>
#include <UniversalTelegramBot.h>


/* ----- Config -------- */ 
const char RST_PIN = D1; 
const char SDA_PIN = D2; 
const int buzzer_pin = D3;
const int led_pin = D4;
const char* ssid = "WiFi";
const char* password = "akash@wifi";

/* Telegram API */
#define botToken "6295891770:AAHlv-TUpbxskHYoganEpU6cHzlyHVtI_Hs"
#define chatId "1087624732"

/* Assign a name to tagid */ 
String getname(String tagid) {
  if (tagid == "e3205f10") { 
    return "Card rfid tag";
  } else if (tagid == "b3fa2513") {
    return "Ring rfid tag";
  } else {
    return tagid;
  }
}

/*------------------------------*/ 


/* Initial Setup */
X509List cert(TELEGRAM_CERTIFICATE_ROOT);
WiFiClientSecure client;
UniversalTelegramBot bot(botToken, client);
MFRC522 mfrc522(SDA_PIN, RST_PIN);  // Create MFRC522 instance

/* Extra Functions */
void indicator(int arg) {
  if (arg == 1) {
    digitalWrite(led_pin, HIGH);
    digitalWrite(buzzer_pin, HIGH);
  } else {
    digitalWrite(led_pin, LOW);
    digitalWrite(buzzer_pin, LOW);
  }
}


void setup() {
  Serial.begin(9600);

  /*Telegram setup*/
  configTime(0, 0, "pool.ntp.org");
  client.setTrustAnchors(&cert);  // Add root certificate for api.telegram.org

  /*Pins*/
  pinMode(led_pin, OUTPUT);
  pinMode(buzzer_pin, OUTPUT);
  digitalWrite(buzzer_pin, HIGH);
  delay(500);
  digitalWrite(buzzer_pin, LOW);

  /*RFID setup*/
  SPI.begin();         // Init SPI bus
  mfrc522.PCD_Init();  // Init MFRC522 card
  Serial.println("RFID reader is ready");

  /*Wifi setup*/
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    digitalWrite(led_pin, HIGH);
    delay(500);
    digitalWrite(led_pin, LOW);
    delay(500);
  }
  Serial.println(WiFi.localIP());
}

void loop() {
  if (mfrc522.PICC_IsNewCardPresent() && mfrc522.PICC_ReadCardSerial()) {
    digitalWrite(buzzer_pin, HIGH);

    String tagId = "";
    for (byte i = 0; i < mfrc522.uid.size; i++) {
      tagId += String(mfrc522.uid.uidByte[i], HEX);
    }
    Serial.println(tagId);
    indicator(1);

    bot.sendMessage(chatId, getname(tagId), "");

    mfrc522.PICC_HaltA();
    delay(200);
    indicator(0);
  }
}
```

<br>

