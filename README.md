# RFID-Activated Automated Light Switch

An embedded systems project built with Arduino that automatically flips a physical room light switch using an RC522 RFID reader and a servo motor. The system uses proximity-based timeout tracking and alternating angles to toggle the physical switch state cleanly.

## 🚀 How It Works (Code Logic)

Unlike simple toggle scripts, this project uses non-blocking timing (`millis()`) to track card presence and door motion:
* **Detection:** When an RFID card or tag is brought near the reader, the system resets its timer continuously.
* **Proximity Trigger:** When the card is moved away (exceeding the 1-second `timeout`), the system triggers the servo action.
* **Alternating States:** The system cycles between two configured switch angles (`angleX` = 120° and `angleY` = 60°), always returning to a neutral resting angle (`restAngle` = 90°) after a 300ms pulse.
* **Debounce / Cooldown:** A 2-second cooldown period prevents accidental double-triggers.

## 🛠️ Bill of Materials (BOM)

* Arduino Uno / Nano
* RC522 RFID Reader Module
* Servo Motor (e.g., SG90 / MG996R)
* Jumper wires & breadboard
* RFID tag / card

## 🔌 Pin Connections

| Component | Pin (Component) | Arduino Pin |
| :--- | :--- | :--- |
| **RC522 RFID** | SDA (SS) | Pin 10 |
| | RST | Pin 9 |
| | MOSI | Pin 11 |
| | MISO | Pin 12 |
| | SCK | Pin 13 |
| | 3.3V | 3.3V Pin |
| | GND | GND Pin |
| **Servo Motor** | Signal (PWM) | Pin 5 |
| | VCC | 5V |
| | GND | GND Pin |

## 💻 Source Code

```cpp
#include <SPI.h>
#include <MFRC522.h>
#include <Servo.h>

#define SS_PIN 10
#define RST_PIN 9

MFRC522 rfid(SS_PIN, RST_PIN);
Servo doorServo;

unsigned long lastSeen = 0;
const unsigned long timeout = 1000;   // ms without a read = switch action
bool cardEverSeen = false;
bool doorIsOpen = false;

unsigned long lastTrigger = 0;
const unsigned long cooldown = 2000;  // won't re-trigger within 2 sec

bool useFirstAngle = true;   // tracks which angle to use next
const int angleX = 120;      // ON position angle
const int angleY = 60;       // OFF position angle
const int restAngle = 90;    // Neutral resting angle

void setup() {
  Serial.begin(9600);
  SPI.begin();
  rfid.PCD_Init();

  doorServo.attach(5);
  doorServo.write(restAngle);

  Serial.println("Ready.");
}

void loop() {
  bool cardSeenThisLoop = rfid.PICC_IsNewCardPresent() && rfid.PICC_ReadCardSerial();

  if (cardSeenThisLoop) {
    lastSeen = millis();
    cardEverSeen = true;
    doorIsOpen = false;
    rfid.PICC_HaltA();
    rfid.PCD_StopCrypto1();
  }

  if (cardEverSeen && !doorIsOpen && (millis() - lastSeen > timeout) && (millis() - lastTrigger > cooldown)) {
    doorIsOpen = true;
    lastTrigger = millis();

    int targetAngle = useFirstAngle ? angleX : angleY;
    Serial.print("Triggered - moving to angle: ");
    Serial.println(targetAngle);

    doorServo.write(targetAngle);
    delay(300);
    doorServo.write(restAngle);

    useFirstAngle = !useFirstAngle;
  }
}
```

## ⚙️ Engineering Highlights

* **Non-Blocking Logic:** Utilized `millis()` instead of `delay()` inside the main loop to keep sensor reading active and responsive.
* **State Management:** Implemented an immediate pulse-and-reset mechanism (`restAngle`) so the servo motor releases force after toggling the wall switch, avoiding continuous current draw and gear strain.


https://github.com/user-attachments/assets/f89f5304-4b10-4c35-8cf9-2f687debb975

