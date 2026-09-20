/*
  AgriBot - Smart Agriculture Rover
  Proposed ESP32 prototype code

  Sensors:
  - Soil moisture sensor: GPIO34 (analog)
  - DHT11/DHT22: GPIO4
  - HC-SR04: TRIG GPIO5, ECHO GPIO18
  - Water pump relay: GPIO23
  - L298N motor driver: GPIO25, GPIO26, GPIO27, GPIO14

  NOTE:
  Calibrate the soil moisture threshold for the actual sensor.
*/

#include <DHT.h>

#define DHTPIN 4
#define DHTTYPE DHT11

#define SOIL_PIN 34
#define TRIG_PIN 5
#define ECHO_PIN 18
#define PUMP_RELAY 23

#define IN1 25
#define IN2 26
#define IN3 27
#define IN4 14

DHT dht(DHTPIN, DHTTYPE);

const int DRY_THRESHOLD = 2200;  // Change after calibration

long readDistanceCM() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);

  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  long duration = pulseIn(ECHO_PIN, HIGH, 30000);
  if (duration == 0) return 999;

  return duration * 0.0343 / 2;
}

void stopMotors() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
}

void moveForward() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void setup() {
  Serial.begin(115200);

  pinMode(SOIL_PIN, INPUT);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(PUMP_RELAY, OUTPUT);

  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  digitalWrite(PUMP_RELAY, LOW);
  stopMotors();

  dht.begin();
}

void loop() {
  int soilValue = analogRead(SOIL_PIN);
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();
  long distance = readDistanceCM();

  Serial.println("----- AgriBot -----");
  Serial.print("Soil: ");
  Serial.println(soilValue);

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println(" C");

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");

  Serial.print("Obstacle distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  // Targeted irrigation
  if (soilValue > DRY_THRESHOLD) {
    digitalWrite(PUMP_RELAY, HIGH);
    Serial.println("Soil is dry -> Pump ON");
  } else {
    digitalWrite(PUMP_RELAY, LOW);
    Serial.println("Soil moisture OK -> Pump OFF");
  }

  // Simple obstacle avoidance
  if (distance < 25) {
    stopMotors();
    Serial.println("Obstacle detected -> Rover stopped");
    delay(1000);
  } else {
    moveForward();
  }

  delay(2000);
}
