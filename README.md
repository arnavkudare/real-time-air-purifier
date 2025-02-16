# real-time-air-purifier
Real time Air purifier using Atmega microcontroller and sensors parallel to the esp-02 Wifi Module
ATMega 32 microcontroller is used for processing the sensors values
16x2 I2C LCD used for displaying of the Live Sensor values and Status of the Air quality
Gas Sensors: MQ2, MQ7 and MQ135 gas Sensors with are electronically adaptible to the latest Arduino board configurations are used for air quality sensing.
In addition to this, to wirelessly transmit the values to the user system, ESP-02 wifi module is used that connects to the kernel of the arduino.
To demonstrate this, we have used a Relay which acts as feedback component is the circuit to turn on the exaust fan which is placed along with filters, and the fans switched on upon crossing the threshold values of the Gas Sensors

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <DHT.h>
#include <SoftwareSerial.h>

#define DHTPIN 2      // DHT11 data pin
#define DHTTYPE DHT11 // DHT11 sensor type

#define MQ7PIN A0     // MQ7 analog pin
#define MQ135PIN A1   // MQ135 analog pin
#define RELAY_PIN 3   // Digital pin connected to relay module

LiquidCrystal_I2C lcd(0x27, 16, 2);  // Address 0x27, 16 columns, 2 rows
DHT dht(DHTPIN, DHTTYPE);
SoftwareSerial espSerial(8, 9); // RX, TX for ESP8266

void setup() {
  Serial.begin(9600);
  espSerial.begin(9600);
  dht.begin();
  lcd.init();
  lcd.backlight();
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Initializing...");
  pinMode(RELAY_PIN, OUTPUT); // Set relay pin as output
  connectWiFi();
}

void loop() {
  float mq7Value = analogRead(MQ7PIN);
  float mq135Value = analogRead(MQ135PIN);
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  // Check sensor values and activate relay if thresholds are exceeded
  if (mq7Value > 150 || mq135Value > 500 || humidity > 65) {
    digitalWrite(RELAY_PIN, HIGH); // Activate relay
    lcd.setCursor(0, 1);
    lcd.print("Fan: ON ");
  } else {
    digitalWrite(RELAY_PIN, LOW); // Deactivate relay
    lcd.setCursor(0, 1);
    lcd.print("Fan: OFF");
  }

  // Display sensor values on LCD
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("CO:");
  lcd.setCursor(0, 1);
  lcd.print("A_Q:");
  lcd.setCursor(4, 0);
  lcd.print(mq7Value);
  lcd.setCursor(4, 1);
  lcd.print(mq135Value);

  lcd.setCursor(0, 0);
  lcd.print("Temp:");
  lcd.setCursor(0, 1);
  lcd.print("Humidity:");
  lcd.setCursor(5, 0);
  lcd.print(temperature);
  lcd.print("C");
  lcd.setCursor(9, 1);
  lcd.print(humidity);
  lcd.print("%");

  // Transmit sensor values wirelessly using ESP01
  transmitData(mq7Value, mq135Value, temperature, humidity);

  delay(2000);
}

void transmitData(float mq7, float mq135, float temp, float hum) {
  espSerial.print("AT+CIPSTART=\"TCP\",\"YOUR_SERVER_IP\",YOUR_PORT\r\n");
  if (espSerial.find("OK")) {
    String sendData = "GET /update?mq7=" + String(mq7) + "&mq135=" + String(mq135) + "&temp=" + String(temp) + "&hum=" + String(hum) + " HTTP/1.1\r\n";
    espSerial.print("AT+CIPSEND=");
    espSerial.println(sendData.length());
    if (espSerial.find(">")) {
      espSerial.print(sendData);
      delay(1000);
      espSerial.println();
      Serial.println("Data Sent");
    }
  }
  espSerial.println("AT+CIPCLOSE");
}

void connectWiFi() {
  espSerial.println("AT+RST");
  if (espSerial.find("OK")) {
    Serial.println("Module Reset");
  }
  delay(1000);

  espSerial.println("AT+CWMODE=1");
  if (espSerial.find("OK")) {
    Serial.println("Station Mode Set");
  }
  delay(1000);

  espSerial.println("AT+CWJAP=\"YOUR_WIFI_SSID\",\"YOUR_WIFI_PASSWORD\"");
  if (espSerial.find("OK")) {
    Serial.println("Connected to WiFi");
  }
  delay(1000);
}


