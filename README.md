# smart-plant-watering-system-
Smart Plant Watering System using Arduino UNO with soil moisture monitoring and automatic water control.
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <DHT.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64

#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// Pins
#define SOIL_PIN A0
#define WATER_PIN A1
#define DHT_PIN 2
#define RELAY_PIN 7
#define BUZZER_PIN 8
#define LED_PIN 9

#define DHTTYPE DHT11
DHT dht(DHT_PIN, DHTTYPE);

void setup() {
  Serial.begin(9600);

  pinMode(RELAY_PIN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(LED_PIN, OUTPUT);

  // Pump OFF initially
  digitalWrite(RELAY_PIN, HIGH);
  digitalWrite(BUZZER_PIN, LOW);
  digitalWrite(LED_PIN, LOW);

  dht.begin();

  // OLED start
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    while (1);
  }

  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);
  display.setTextSize(1);
  display.setCursor(15, 25);
  display.println("Smart Plant");
  display.display();

  delay(2000);
}

void loop() {

  int soilValue = analogRead(SOIL_PIN);
  int waterValue = analogRead(WATER_PIN);

  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  // Convert soil value to percentage
  int soilPercent = map(soilValue, 1023, 0, 0, 100);
  soilPercent = constrain(soilPercent, 0, 100);

  // Display
  display.clearDisplay();
  display.setCursor(0, 0);

  display.print("Soil: ");
  display.print(soilPercent);
  display.println("%");

  display.print("Water: ");
  display.println(waterValue);

  display.print("Temp: ");
  display.print(temperature);
  display.println(" C");

  display.print("Humidity: ");
  display.print(humidity);
  display.println("%");

  // Watering condition
  if (soilPercent < 30) {

    // Pump ON
    digitalWrite(RELAY_PIN, LOW);
    digitalWrite(LED_PIN, HIGH);
    digitalWrite(BUZZER_PIN, HIGH);

    display.println("Pump: ON");

  } else {

    // Pump OFF
    digitalWrite(RELAY_PIN, HIGH);
    digitalWrite(LED_PIN, LOW);
    digitalWrite(BUZZER_PIN, LOW);

    display.println("Pump: OFF");
  }

  display.display();

  delay(2000);
}
