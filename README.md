# Elderly-Helper
I made a new elderly tool
Heres the code for it
#include <Wire.h>
#include <MPU6050.h>

MPU6050 mpu;

const int trigPin = 6;
const int echoPin = 7;
const int ledPin = 3;
const int buzzerPin = 4;
const int buttonPin = 2;

bool fallDetected = false;
bool monitoringStillness = false;
unsigned long fallTime = 0;
unsigned long ledTime = 0;
unsigned long lastDistance = 0;

void setup() {
  Serial.begin(9600);
  Wire.begin();
  mpu.initialize();

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(ledPin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);
  pinMode(buttonPin, INPUT_PULLUP);

  digitalWrite(ledPin, LOW);
  digitalWrite(buzzerPin, LOW);
}

void loop() {
  if (!fallDetected) {
    // Check vibration
    int16_t ax, ay, az;
    mpu.getAcceleration(&ax, &ay, &az);
    long acc = abs(ax) + abs(ay) + abs(az);

    if (acc > 25000) {  // threshold for sudden fall
      Serial.println("Fall Detected!");
      fallDetected = true;
      monitoringStillness = true;
      fallTime = millis();
      lastDistance = getDistance();
    }
  }

  if (monitoringStillness) {
    // Check if person has moved after falling
    long currentTime = millis();
    long dist = getDistance();
    Serial.print("Distance: "); Serial.println(dist);

    // Check if distance has changed significantly
    if (abs(dist - lastDistance) > 5) {
      Serial.println("Movement Detected, cancelling...");
      resetAll();
      return;
    }

    // If stayed in same spot for 30 secs
    if (currentTime - fallTime >= 30000) {
      digitalWrite(ledPin, HIGH);
      ledTime = currentTime;
      monitoringStillness = false; // stop monitoring, we already triggered LED
    }
  }

  // After LED on for 5 seconds, start buzzer
  if (ledTime > 0 && millis() - ledTime >= 5000) {
    digitalWrite(buzzerPin, HIGH);
  }

  // Push button to cancel alarm
  if (digitalRead(buttonPin) == LOW) {
    Serial.println("Reset button pressed");
    resetAll();
  }

  delay(500); // half sec delay to chill
}

long getDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  long duration = pulseIn(echoPin, HIGH);
  return duration * 0.034 / 2;
}

void resetAll() {
  fallDetected = false;
  monitoringStillness = false;
  ledTime = 0;
  digitalWrite(ledPin, LOW);
  digitalWrite(buzzerPin, LOW);
}

and the wiring is simple 
MPU-6050
VCC - 5V
GND - GND
SDA - A4
SCL - A5

Ultrasonic Sensor (HC-SR04)
VCC - 5V
GND - GND
Trig - D6
Echo - D7

LED
Anode  - D3
Cathode  - GND

Buzzer
D4
GND

Push Button
One side - D2
Other side - GND
