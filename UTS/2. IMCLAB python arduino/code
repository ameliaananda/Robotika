#include <Arduino.h>
#include <stdlib.h>
#include <string.h>

// iMCLab motor pins retained from your supplied example.
const int ledPin = 2;
const int motor1Pin1 = 27;
const int motor1Pin2 = 26;
const int enable1Pin = 12;
const int freq = 30000;
const int resolution = 8;
const int DEFAULT_DUTY = 200;
const int DUTY_STEP = 10;
const unsigned long COMMAND_TIMEOUT_MS = 2000;

int dutyCycle = 0;
bool ledOn = false;
bool ready = false;
const char *fault = "PWM_SETUP";
unsigned long lastCommandMs = 0;
unsigned long lastReportMs = 0;

void reportState() {
  Serial.printf("STATE,PWM,%d,%d,%s\n", dutyCycle, ledOn ? 1 : 0, fault);
}

void applyMotor(int duty) {
  dutyCycle = constrain(duty, 0, 300);
  digitalWrite(motor1Pin1, dutyCycle > 0 ? HIGH : LOW);
  digitalWrite(motor1Pin2, LOW);
  ledcWrite(enable1Pin, dutyCycle);
  // Drive indicator only: without sensor feedback this does not prove rotation.
  ledOn = dutyCycle > 0;
  digitalWrite(ledPin, ledOn ? HIGH : LOW);
}

void acceptCommand(const char *line) {
  int requested;
  if (strcmp(line, "0") == 0) {
    requested = 0;
  } else if (strcmp(line, "1") == 0) {
    requested = dutyCycle > 0 ? dutyCycle : DEFAULT_DUTY;
  } else if (strcmp(line, "+") == 0) {
    requested = constrain(dutyCycle + DUTY_STEP, 0, 255);
  } else if (strcmp(line, "-") == 0) {
    requested = constrain(dutyCycle - DUTY_STEP, 0, 255);
  } else if (strncmp(line, "PWM:", 4) == 0) {
    char *end;
    long value = strtol(line + 4, &end, 10);
    if (end == line + 4 || *end != '\0' || value < 0 || value > 255) {
      Serial.println("ERR PWM must be an integer 0..255");
      return;
    }
    requested = (int)value;
  } else {
    Serial.println("ERR use 1, 0, +, -, or PWM:0..255");
    return;
  }

  if (!ready) {
    Serial.println("ERR PWM_SETUP");
    return;
  }
  if (requested > 0 && strcmp(fault, "NONE") != 0) {
    Serial.println("ERR send 0 or PWM:0 to clear fault first");
    return;
  }

  applyMotor(requested);
  if (requested == 0) fault = "NONE";
  lastCommandMs = millis();
  reportState();
}

void readCommands() {
  static char line[32];
  static unsigned int used = 0;
  static bool overflow = false;

  for (int budget = 0; budget < 64 && Serial.available(); ++budget) {
    char c = (char)Serial.read();
    if (c == '\r' || c == '\n') {
      line[used] = '\0';
      if (overflow) Serial.println("ERR command too long");
      else if (used) acceptCommand(line);
      used = 0;
      overflow = false;
    } else if (c == '\0' || used >= sizeof(line) - 1) {
      overflow = true;
    } else if (!overflow) {
      line[used++] = c;
    }
  }
}

void setup() {
  Serial.begin(115200);
  digitalWrite(enable1Pin, LOW);
  pinMode(enable1Pin, OUTPUT);
  pinMode(motor1Pin1, OUTPUT);
  pinMode(motor1Pin2, OUTPUT);
  pinMode(ledPin, OUTPUT);
  digitalWrite(motor1Pin1, LOW);
  digitalWrite(motor1Pin2, LOW);
  digitalWrite(ledPin, LOW);

  // ESP32 core 3.x: pin-based PWM replaces ledcSetup/ledcAttachPin.
  if (!ledcAttach(enable1Pin, freq, resolution)) {
    Serial.println("ERR PWM_SETUP");
    reportState();
    return;
  }
  ready = true;
  fault = "NONE";
  applyMotor(0);
  Serial.println("READY PWM");
  reportState();
}

void loop() {
  readCommands();
  unsigned long now = millis();

  // Jupyter repeats an absolute PWM command every 0.5 seconds.
  if (ready && dutyCycle > 0 && now - lastCommandMs >= COMMAND_TIMEOUT_MS) {
    applyMotor(0);
    fault = "COMMAND_TIMEOUT";
    reportState();
  }

  if (now - lastReportMs >= 1000) {
    lastReportMs = now;
    reportState();
  }
}
