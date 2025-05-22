# Smart-gloves-for-speech-disabled-person
#include <Wire.h>
#include <Adafruit_ADXL345_U.h>
#include <LiquidCrystal_I2C.h>

#define MODE_SWITCH_PIN 2
#define FAN_RELAY_PIN 8
#define LIGHT_RELAY_PIN 9
#define VOICE_MSG1_PIN 10
#define VOICE_MSG2_PIN 11

Adafruit_ADXL345_Unified accel = Adafruit_ADXL345_Unified(123);
LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  Serial.begin(9600);
  accel.begin();
  lcd.begin();
  lcd.backlight();

  pinMode(MODE_SWITCH_PIN, INPUT_PULLUP);
  pinMode(FAN_RELAY_PIN, OUTPUT);
  pinMode(LIGHT_RELAY_PIN, OUTPUT);
  pinMode(VOICE_MSG1_PIN, OUTPUT);
  pinMode(VOICE_MSG2_PIN, OUTPUT);

  lcd.setCursor(0, 0);
  lcd.print("Smart Glove Ready");
  delay(2000);
  lcd.clear();
}

void loop() {
  sensors_event_t event;
  accel.getEvent(&event);

  bool isVoiceMode = digitalRead(MODE_SWITCH_PIN) == LOW;

  lcd.setCursor(0, 0);
  lcd.print(isVoiceMode ? "Voice Mode     " : "Device Mode    ");

  // Example gesture conditions
  if (event.acceleration.x > 7) {
    if (isVoiceMode) {
      playVoiceMessage1();
    } else {
      digitalWrite(FAN_RELAY_PIN, HIGH);
      lcd.setCursor(0, 1);
      lcd.print("Fan ON         ");
    }
  } else if (event.acceleration.y > 7) {
    if (isVoiceMode) {
      playVoiceMessage2();
    } else {
      digitalWrite(LIGHT_RELAY_PIN, HIGH);
      lcd.setCursor(0, 1);
      lcd.print("Light ON       ");
    }
  } else {
    // Turn off devices if gesture not held
    digitalWrite(FAN_RELAY_PIN, LOW);
    digitalWrite(LIGHT_RELAY_PIN, LOW);
    lcd.setCursor(0, 1);
    lcd.print("Devices OFF    ");
  }

  delay(500);
}

void playVoiceMessage1() {
  digitalWrite(VOICE_MSG1_PIN, HIGH);
  delay(500);
  digitalWrite(VOICE_MSG1_PIN, LOW);
  lcd.setCursor(0, 1);
  lcd.print("I need water   ");
}

void playVoiceMessage2() {
  digitalWrite(VOICE_MSG2_PIN, HIGH);
  delay(500);
  digitalWrite(VOICE_MSG2_PIN, LOW);
  lcd.setCursor(0, 1);
  lcd.print("I need help    ");
}
