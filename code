#include <LiquidCrystal_I2C.h>
#define BLYNK_PRINT Serial
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <DHT.h>

// Initialize the LCD display
LiquidCrystal_I2C lcd(0x27, 16, 2);

char auth[] = "1WkKP1Tg3TQ28zZCYtbxqviy0e9qbUVi";  // Enter your Blynk Auth token
char ssid[] = "Sanjana";         // Enter your WIFI SSID
char pass[] = "98463060";     // Enter your WIFI Password

DHT dht(D4, DHT11);  // D4 for DHT11 Temperature Sensor

#define soil A0       // A0 Soil Moisture Sensor
#define relayPin D7   // D3 Relay Pin

bool isMotorOn = false;

BlynkTimer timer;

void setup() {
  Serial.begin(9600);
  lcd.begin();
  lcd.backlight();
  pinMode(relayPin, OUTPUT);
  digitalWrite(relayPin, LOW);

  Blynk.begin(auth, ssid, pass, "blynk.cloud", 80);
  dht.begin();

  lcd.setCursor(0, 0);
  lcd.print("  Initializing  ");
  for (int a = 5; a <= 10; a++) {
    lcd.setCursor(a, 1);
    lcd.print(".");
    delay(500);
  }
  lcd.clear();
  lcd.setCursor(11, 1);
  lcd.print("M:OFF");

  timer.setInterval(100L, soilMoistureSensor);
  timer.setInterval(100L, DHT11sensor);
}

void DHT11sensor() {
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }
  Blynk.virtualWrite(V0, t);
  Blynk.virtualWrite(V1, h);

  lcd.setCursor(0, 0);
  lcd.print("T:");
  lcd.print(t);

  lcd.setCursor(8, 0);
  lcd.print("H:");
  lcd.print(h);
}

void soilMoistureSensor() {
  int value = analogRead(soil);
  value = map(value, 0, 1024, 0, 100);
  value = (value - 100) * -1;

  Blynk.virtualWrite(V2, value);
  lcd.setCursor(0, 1);
  lcd.print("S:");
  lcd.print(value);
  lcd.print(" ");

  if (value > 40) { // Adjust the threshold as needed
    digitalWrite(relayPin, HIGH);
    if (!isMotorOn) {
      Blynk.logEvent("motor_off", "Motor is turned off!"); // Send notification when motor is turned on
      isMotorOn = true;
    }
    lcd.setCursor(11, 1);
    lcd.print("M:OFF"); // Turn on the motor
  } else {
    digitalWrite(relayPin, LOW);
    isMotorOn = false;
    Blynk.logEvent("motor_on", "Motor is turned on!");
    lcd.setCursor(11, 1);
    lcd.print("M:ON"); // Turn off the motor
  }
}

void loop() {
  Blynk.run();
  timer.run();
}
