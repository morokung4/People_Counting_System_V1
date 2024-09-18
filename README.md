#define TRIGGER_PIN_1 2
#define ECHO_PIN_1 3
#define TRIGGER_PIN_2 4
#define ECHO_PIN_2 5

int peopleCount = 0;
bool sensor1Triggered = false;
bool sensor2Triggered = false;
unsigned long lastTriggerTime = 0;
const unsigned long RESET_TIME = 1000; // 1 second

void setup() {
  Serial.begin(9600);
  pinMode(TRIGGER_PIN_1, OUTPUT);
  pinMode(ECHO_PIN_1, INPUT);
  pinMode(TRIGGER_PIN_2, OUTPUT);
  pinMode(ECHO_PIN_2, INPUT);
}

void loop() {
  long distance1 = measureDistance(TRIGGER_PIN_1, ECHO_PIN_1);
  long distance2 = measureDistance(TRIGGER_PIN_2, ECHO_PIN_2);

  // 30 cen distance
  if (distance1 < 30 && !sensor1Triggered) {
    sensor1Triggered = true;
    lastTriggerTime = millis();
  }

  if (distance2 < 30 && !sensor2Triggered) {
    sensor2Triggered = true;
    lastTriggerTime = millis();
  }

  // Check if both sensors were triggered
  if (sensor1Triggered && sensor2Triggered) {
    if (distance1 < distance2) {
      peopleCount++;
      Serial.println("Person entered. Count: " + String(peopleCount));
    } else {
      peopleCount = max(0, peopleCount - 1);
      Serial.println("Person exited. Count: " + String(peopleCount));
    }
    resetSensors();
  }

  // Reset sensors if too much time has passed
  if (millis() - lastTriggerTime > RESET_TIME) {
    resetSensors();
  }

  delay(100); // adjust for fine tuning
}

long measureDistance(int triggerPin, int echoPin) {
  digitalWrite(triggerPin, LOW);
  delayMicroseconds(2);
  digitalWrite(triggerPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(triggerPin, LOW);
  
  long duration = pulseIn(echoPin, HIGH);
  return duration * 0.034 / 2;
}

void resetSensors() {
  sensor1Triggered = false;
  sensor2Triggered = false;
}
