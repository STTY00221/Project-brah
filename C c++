#include <Arduino.h>
#include <HX711.h>
#include <LiquidCrystal_I2C.h>
#include <WiFi.h>
#include <HTTPClient.h>

// กำหนดพินต่างๆ
#define LOADCELL_DOUT_PIN  D3
#define LOADCELL_SCK_PIN   D4
#define SENSOR_PIN         D5
#define LED_GREEN_PIN      D6
#define LED_YELLOW_PIN     D7
#define LED_RED_PIN        D8
#define BUTTON_PIN         D9

// กำหนดค่าต่างๆ
#define WIFI_SSID       "group_2"
#define WIFI_PASSWORD   "!project001"
#define LINE_API_URL    "https://notify-api.line.me/api/notify"
#define LINE_TOKEN      "oQ0YYWDDBlVreuJW7mwTerJJT6JwQOgayASuQET0Sur"

// สร้างออบเจ็กต์
HX711 scale;
LiquidCrystal_I2C lcd(0x27, 16, 2);  // ปรับที่อยู่ I2C ของ LCD หากจำเป็น

// ตัวแปรต่างๆ
float calibration_factor = -7050; // ปรับค่านี้ตามการสอบเทียบ
float weight_threshold_full = 15.0; // กิโลกรัม
float weight_threshold_low = 5.0; // กิโลกรัม
int buttonState = 0;
bool objectDetected = false;

unsigned long lastResetTime = 0;
bool resetCountdownStarted = false;

void setup() {
  Serial.begin(115200);
  lcd.init();
  lcd.backlight();

  pinMode(SENSOR_PIN, INPUT);
  pinMode(LED_GREEN_PIN, OUTPUT);
  pinMode(LED_YELLOW_PIN, OUTPUT);
  pinMode(LED_RED_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);

  scale.begin(LOADCELL_DOUT_PIN, LOADCELL_SCK_PIN);
  scale.set_scale(calibration_factor);
  scale.tare();

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
}

void loop() {
  objectDetected = digitalRead(SENSOR_PIN) == HIGH;

  if (objectDetected) {
    float weight = scale.get_units(10);
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Weight: ");
    lcd.print(weight);
    lcd.print(" kg");

    // ควบคุมไฟ LED ตามน้ำหนัก
    if (weight >= weight_threshold_full) {
      digitalWrite(LED_GREEN_PIN, HIGH);
      digitalWrite(LED_YELLOW_PIN, LOW);
      digitalWrite(LED_RED_PIN, LOW);
    } else if (weight >= weight_threshold_low) {
      digitalWrite(LED_GREEN_PIN, LOW);
      digitalWrite(LED_YELLOW_PIN, HIGH);
      digitalWrite(LED_RED_PIN, LOW);
    } else {
      digitalWrite(LED_GREEN_PIN, LOW);
      digitalWrite(LED_YELLOW_PIN, LOW);
      digitalWrite(LED_RED_PIN, HIGH);
    }

    // ตรวจสอบการกดปุ่มรีเซ็ต
    buttonState = digitalRead(BUTTON_PIN);
    if (buttonState == LOW) {
      scale.tare();
      lastResetTime = millis(); 
      resetCountdownStarted = true;
      lcd.clear();
      lcd.print("Reset done!");
      delay(1000);
    }

    // แสดงผล LCD ก่อนรีเซ็ต
    if (resetCountdownStarted) {
      unsigned long timeSinceReset = millis() - lastResetTime;
      if (timeSinceReset >= 20 * 60 * 1000) { // 20 นาที
        // รีเซ็ตอัตโนมัติ
        scale.tare();
        lcd.clear();
        lcd.print("Reset done!");
        resetCountdownStarted = false;
        delay(1000);
      } else if (timeSinceReset >= 19 * 60 * 1000) { // แสดงผลก่อน 1 นาที
        int secondsRemaining = (20 * 60 * 1000 - timeSinceReset) / 1000;
        lcd.clear();
        lcd.print("Auto reset in ");
        lcd.print(secondsRemaining);
        lcd.print("s");
      }
    }
  } else {
    lcd.clear();
    lcd.print("No object");
    digitalWrite(LED_GREEN_PIN, LOW);
    digitalWrite(LED_YELLOW_PIN, LOW);
    digitalWrite(LED_RED_PIN, LOW);
  }

  // ส่งข้อความ Line เมื่อมีการสั่งซื้อ (สมมติว่ามีการสั่งซื้อเมื่อน้ำหนักลดลงถึงระดับหนึ่ง)
  if (objectDetected && scale.get_units(10) < weight_threshold_low) {
    sendLineNotification("ได้รับการสั่งซื้อถังแก๊ส!");
  }

  delay(1000);
}

void sendLineNotification(String message) {
  HTTPClient http;
  http.begin(LINE_API_URL);
  http.addHeader("Content-Type", "application/x-www-form-urlencoded");

  String payload = "message=" + message;
  int httpResponseCode = http.POST(payload);

  if (httpResponseCode > 0) {
    Serial.print("Line notification sent, HTTP response code: ");
    Serial.println(httpResponseCode);
  } else {
    Serial.print("Error sending Line notification, error: ");
    Serial.println(http.errorToString(httpResponseCode));
  }

  http.end();
}
