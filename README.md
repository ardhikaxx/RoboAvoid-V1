# RoboAvoid-V1

## Deskripsi

**RoboAvoid-V1** adalah robot pencegah tabrakan yang menggunakan sensor ultrasonic untuk mendeteksi rintangan di depannya dan menghindarinya. Robot ini dirancang untuk bergerak maju, mundur, atau berbelok kiri/kanan tergantung pada jarak dari rintangan yang terdeteksi. Status pergerakan robot ditampilkan pada LCD 16x2 I2C.

## Komponen

- **Wemos D1 Mini** (sebagai mikrokontroler)
- **HC-SR04 Ultrasonic Sensor** (untuk deteksi jarak)
- **L298N H-Bridge Motor Driver** (untuk mengendalikan motor)
- **LCD 16x2 I2C** (untuk menampilkan status)

## Skema Pin

### Wemos D1 Mini:
- **TRIGGER_PIN**: D3 (GPIO 0) - Pin untuk mengaktifkan sensor ultrasonic.
- **ECHO_PIN**: D4 (GPIO 2) - Pin untuk menerima sinyal dari sensor ultrasonic.
- **MOTOR_LEFT_FORWARD**: D5 (GPIO 14) - Pin untuk menggerakkan motor kiri maju.
- **MOTOR_LEFT_BACKWARD**: D6 (GPIO 12) - Pin untuk menggerakkan motor kiri mundur.
- **MOTOR_RIGHT_FORWARD**: D7 (GPIO 13) - Pin untuk menggerakkan motor kanan maju.
- **MOTOR_RIGHT_BACKWARD**: D8 (GPIO 15) - Pin untuk menggerakkan motor kanan mundur.

### HC-SR04 Ultrasonic Sensor:
- **VCC**: 5V (Wemos D1 Mini)
- **GND**: GND (Wemos D1 Mini)
- **TRIG**: D3 (Wemos D1 Mini)
- **ECHO**: D4 (Wemos D1 Mini)

### L298N H-Bridge Motor Driver:
- **IN1**: D5 (Wemos D1 Mini) - Motor kiri maju
- **IN2**: D6 (Wemos D1 Mini) - Motor kiri mundur
- **IN3**: D7 (Wemos D1 Mini) - Motor kanan maju
- **IN4**: D8 (Wemos D1 Mini) - Motor kanan mundur
- **ENA**: (Enable pin untuk motor kiri) - Dihubungkan ke 5V atau PWM untuk pengendalian kecepatan
- **ENB**: (Enable pin untuk motor kanan) - Dihubungkan ke 5V atau PWM untuk pengendalian kecepatan
- **VCC**: Catu daya motor (misalnya 12V)
- **GND**: Ground

### LCD 16x2 I2C:
- **VCC**: 5V (Wemos D1 Mini)
- **GND**: GND (Wemos D1 Mini)
- **SDA**: D1 (Wemos D1 Mini)
- **SCL**: D2 (Wemos D1 Mini)

## Kode

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <NewPing.h>

// Pin setup
#define TRIGGER_PIN D3
#define ECHO_PIN D4
#define MOTOR_LEFT_FORWARD D5
#define MOTOR_LEFT_BACKWARD D6
#define MOTOR_RIGHT_FORWARD D7
#define MOTOR_RIGHT_BACKWARD D8

// LCD setup
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Sensor setup
NewPing sonar(TRIGGER_PIN, ECHO_PIN, 200); // Max distance is 200 cm

// Motor speed
#define MOTOR_SPEED 255

void setup() {
  lcd.begin();
  lcd.backlight();
  pinMode(MOTOR_LEFT_FORWARD, OUTPUT);
  pinMode(MOTOR_LEFT_BACKWARD, OUTPUT);
  pinMode(MOTOR_RIGHT_FORWARD, OUTPUT);
  pinMode(MOTOR_RIGHT_BACKWARD, OUTPUT);

  // Show startup animation
  startupAnimation();
}

void loop() {
  delay(50);
  unsigned int distance = sonar.ping_cm();

  if (distance == 0 || distance > 20) {
    moveForward();
  } else if (distance <= 20 && distance > 10) {
    stopMovement();
    lcd.setCursor(0, 0);
    lcd.print("Mundur      ");
    delay(500);
    moveBackward();
  } else {
    stopMovement();
    if (random(0, 2) == 0) {
      lcd.setCursor(0, 0);
      lcd.print("Belok Kanan ");
      turnRight();
    } else {
      lcd.setCursor(0, 0);
      lcd.print("Belok Kiri  ");
      turnLeft();
    }
  }
}

void startupAnimation() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Menghidupkan");
  
  // Animation 1: Scrolling text
  for (int i = 0; i < 16; i++) {
    lcd.setCursor(i, 1);
    lcd.print("RoboAvoid-V1");
    delay(150);
  }
  
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("RoboAvoid-V1");
  lcd.setCursor(0, 1);
  lcd.print("Siap Beraksi!");
  delay(1000);

  // Animation 2: Blinking dots with different speeds
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("      .");
  delay(200);
  lcd.setCursor(0, 0);
  lcd.print("     ..");
  delay(150);
  lcd.setCursor(0, 0);
  lcd.print("    ...");
  delay(100);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("      .");
  delay(200);
}

void moveForward() {
  digitalWrite(MOTOR_LEFT_FORWARD, HIGH);
  digitalWrite(MOTOR_LEFT_BACKWARD, LOW);
  digitalWrite(MOTOR_RIGHT_FORWARD, HIGH);
  digitalWrite(MOTOR_RIGHT_BACKWARD, LOW);
  lcd.setCursor(0, 0);
  lcd.print("Maju        ");
}

void moveBackward() {
  digitalWrite(MOTOR_LEFT_FORWARD, LOW);
  digitalWrite(MOTOR_LEFT_BACKWARD, HIGH);
  digitalWrite(MOTOR_RIGHT_FORWARD, LOW);
  digitalWrite(MOTOR_RIGHT_BACKWARD, HIGH);
  lcd.setCursor(0, 0);
  lcd.print("Mundur      ");
}

void turnRight() {
  digitalWrite(MOTOR_LEFT_FORWARD, HIGH);
  digitalWrite(MOTOR_LEFT_BACKWARD, LOW);
  digitalWrite(MOTOR_RIGHT_FORWARD, LOW);
  digitalWrite(MOTOR_RIGHT_BACKWARD, HIGH);
}

void turnLeft() {
  digitalWrite(MOTOR_LEFT_FORWARD, LOW);
  digitalWrite(MOTOR_LEFT_BACKWARD, HIGH);
  digitalWrite(MOTOR_RIGHT_FORWARD, HIGH);
  digitalWrite(MOTOR_RIGHT_BACKWARD, LOW);
}

void stopMovement() {
  digitalWrite(MOTOR_LEFT_FORWARD, LOW);
  digitalWrite(MOTOR_LEFT_BACKWARD, LOW);
  digitalWrite(MOTOR_RIGHT_FORWARD, LOW);
  digitalWrite(MOTOR_RIGHT_BACKWARD, LOW);
}
