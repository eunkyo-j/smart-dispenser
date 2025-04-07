# 💧 스마트 정수기 with Arduino

## 📌 프로젝트 개요  
건강 관리가 중요한 현대인을 위한 **자동 복약 & 정수 시스템**입니다.  
지정된 시간에 알람이 울리고, 사람이 다가오면 자동으로 **영양제/약을 손 위에 떨어뜨리고**, **물까지 따라주는 정수기**입니다.  
외형은 **우드락을 직접 잘라 조립해 구현**하였으며, 하드웨어와 소프트웨어를 모두 직접 설계하고 개발하였습니다.

---

## ⚙️ 주요 기능

1. ⏰ **알람 기능** – 사용자가 설정한 시간이 되면 부저가 울림  
2. 🧍 **인체 감지 센서 (PIR)** – 사람이 접근하면 알람이 자동 종료  
3. 💊 **약 자동 분배** – 8등분된 회전 약통에서 서보 모터로 약 1정 배출  
4. 🥤 **컵 인식** – 초음파 센서로 컵을 감지하면 물(150ml) 자동 제공  
5. 🖥️ **LCD 시계** – 현재 시각 표시

---

## 🔧 사용 부품

| 부품 | 용도 |
|------|------|
| Arduino Uno | 메인 컨트롤러 |
| RTC 모듈 | 시간 저장 및 알람 트리거 |
| 피에조 부저 | 알람 출력 |
| PIR 센서 | 사람 감지 |
| 서보 모터 | 회전 약통 제어 (8등분, 45도씩 회전) |
| 초음파 센서 | 컵 감지 |
| 워터펌프 | 물 공급 |
| LCD I2C | 시간 표시용 디스플레이 |

---

## 🧠 개발 포인트

- **센서/모터/디스플레이 연동**을 통해 하드웨어 기반 자동화 구현  
- **우드락을 활용한 외형 제작**으로 하드웨어 제작까지 직접 수행  
- **생활 속 문제 해결 중심의 프로젝트** 기획 및 설계  
- **Arduino C++ 기반 프로그래밍** 경험

---

## 📷 프로젝트 이미지

> 아래 이미지는 예시입니다. 실제 프로젝트 이미지로 교체해주세요!

![정수기 외형 이미지](./img/smart_dispenser.jpg)

---

## 💻 주요 코드 (Arduino)

```cpp
#include <Servo.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include "RTClib.h"

#define buzzerPin 2
#define pirPin 3
#define servoPin 9
#define trigPin 10
#define echoPin 11
#define pumpPin 12

RTC_DS1307 rtc;
LiquidCrystal_I2C lcd(0x27, 16, 2);
Servo servo;

bool alarmTriggered = false;
bool pillDropped = false;
int pillIndex = 0;

void setup() {
  pinMode(buzzerPin, OUTPUT);
  pinMode(pirPin, INPUT);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(pumpPin, OUTPUT);

  servo.attach(servoPin);
  servo.write(0);

  Serial.begin(9600);
  lcd.init();
  lcd.backlight();
  rtc.begin();
}

void loop() {
  DateTime now = rtc.now();

  lcd.setCursor(0, 0);
  lcd.print("Time: ");
  lcd.print(now.hour());
  lcd.print(":");
  lcd.print(now.minute());

  if (now.hour() == 9 && now.minute() == 0 && !alarmTriggered) {
    digitalWrite(buzzerPin, HIGH);
    alarmTriggered = true;
  }

  if (digitalRead(pirPin) == HIGH && alarmTriggered) {
    digitalWrite(buzzerPin, LOW);

    if (!pillDropped) {
      pillIndex = (pillIndex + 1) % 8;
      servo.write(pillIndex * 45);
      delay(1000);
      pillDropped = true;
    }

    long duration, distance;
    digitalWrite(trigPin, LOW);
    delayMicroseconds(2);
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);
    duration = pulseIn(echoPin, HIGH);
    distance = duration * 0.034 / 2;

    if (distance < 10) {
      digitalWrite(pumpPin, HIGH);
      delay(3000);
      digitalWrite(pumpPin, LOW);
    }
  }

  delay(1000);
}
