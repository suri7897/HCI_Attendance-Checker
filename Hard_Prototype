#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
  #include <avr/power.h>
#endif
#include <ESP8266WiFi.h>
#include <WiFiUdp.h>
#include <time.h>

#include <Wire.h>
#include <PN532_I2C.h>
#include <PN532.h>
#include <NfcAdapter.h>

#define SDA_PIN D14
#define SCL_PIN D15
TwoWire myWire = TwoWire();
PN532_I2C pn532_i2c(myWire);
NfcAdapter nfc = NfcAdapter(pn532_i2c);
String tagId = "None";
byte nuidPICC[4];

// LED 설정
#define LED_PIN D6
#define NUMPIXELS 5
#define LEDCOUNT 5
#define BRIGHTNESS 50

#define MOTORPIN D7

// Wi-Fi 설정
const char* ssid = "TP-Link_DFAA";
const char* password = "13865719";

// NTP 서버 설정
const char* ntpServer = "pool.ntp.org";
const long gmtOffset_sec = 32400;  // 한국 시간대 (UTC+9)
const int daylightOffset_sec = 0;

Adafruit_NeoPixel strip = Adafruit_NeoPixel(NUMPIXELS, LED_PIN, NEO_GRB + NEO_KHZ800);


// 전역 변수
bool is_checked = false;
bool is_alerted = false;
int time_minute = 18 * 60 + 10;

void setup() {
  Serial.begin(9600);
  myWire.begin(SDA_PIN, SCL_PIN);
  nfc.begin();
  delay(5000);
  
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected.");
  configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
  
  strip.setBrightness(BRIGHTNESS);
  strip.begin();
  strip.show();

  showFailure();
  pinMode(MOTORPIN,OUTPUT);
}

void loop() {
  delay(100);
  //printLocalTime();  // 시간 출력
  checkRFIDAndTime(); // RFID 검사 및 시간 확인
}

void printLocalTime() {
  struct tm timeinfo;
  if (!getLocalTime(&timeinfo)) {
    Serial.println("Failed to obtain time");
    return;
  }
}

void checkRFIDAndTime() {
  struct tm timeinfo;
  if (getLocalTime(&timeinfo)) {
    int current_minutes = timeinfo.tm_hour * 60 + timeinfo.tm_min;  // 현재 분 계산
    if(!is_alerted && (time_minute - current_minutes)<2){
      Serial.println("Alerted");
      showAlert();
      is_alerted = true;
      showFailure();
    }
    
    if (!is_checked && nfc.tagPresent()) {
      is_checked = true;  // 카드 인식 상태를 true로 변경
      int start_minutes = time_minute - 10;  // 시작 시간 분
      int end_minutes = time_minute + 10;  // 종료 시간 분
      int late_minutes = time_minute + 30;  // 지각 시간 분
      if (current_minutes >= start_minutes && current_minutes <= end_minutes) {
        Serial.println("attendance checked");
        showSuccess();
      } else if(current_minutes > end_minutes && current_minutes <= late_minutes){
        Serial.println("late");
        showLate();
        delay(5000);
        showSuccess();
      }else{
        Serial.println("not in time");
      }
    }

    if (is_checked && time_minute + 75 <= current_minutes) {
      Serial.println("motor vibe");
      showFailure();
      is_alerted = false;
      is_checked = false;  // 다시 체크 안된 상태로 변경
    }
  }
  delay(500);
}

void showSuccess() {
  for(uint16_t i = 0; i < LEDCOUNT ; i++)
    strip.setPixelColor(i, strip.Color(0, 30, 0)); // 초록색
  strip.show();
}

void showFailure() {
  for(uint16_t i = 0; i < LEDCOUNT ; i++)
    strip.setPixelColor(i, strip.Color(30, 0, 0)); // 빨간색
  strip.show();
}

void showAlert() {
  for (int j = 0; j < 10; j++) {
    for(uint16_t i = 0; i < LEDCOUNT; i++)
      strip.setPixelColor(i, strip.Color(255, 0, 0)); // 밝은 빨간색
    strip.show();
    digitalWrite(MOTORPIN, HIGH);
    delay(500);
    strip.clear();
    strip.show();
    digitalWrite(MOTORPIN, LOW);
    delay(500);
  }
}

void showLate() {
  for(uint16_t i = 0; i < LEDCOUNT ; i++)
    strip.setPixelColor(i, strip.Color(30, 30, 0)); // 노란색
  strip.show();
}
