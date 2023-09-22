#include <Arduino.h>
#include <ESP32Servo.h>  // Library for controlling Servo motor
#include <SPI.h>         // SPI communication library
#include <MFRC522.h>     // Library for the MFRC522 RFID module

#define SS_PIN SDA        // SS_PIN is defined as SDA
#define RST_PIN 27        // RST_PIN is defined as 27
#define LED_G 13           // Green LED pin
#define LED_R 15           // Red LED pin
#define BUZZER 2           // Buzzer pin

MFRC522 mfrc522(SS_PIN, RST_PIN); // Create MFRC522 instance
Servo myServo;                     // Define servo name

int generatedOTP = 0;  // Variable to store generated OTP
int greenLEDPin = 13; // Green LED pin
int redLEDPin = 15;   // Red LED pin
int buzzerPin = 2;    // Buzzer pin
int tries = 0;         // Counter for OTP verification attempts

void setup() {
  Serial.begin(115200); // Initialize serial communication at 115200 baud
  randomSeed(analogRead(0)); // Initialize random number generator
  pinMode(greenLEDPin, OUTPUT); // Set green LED pin as output
  pinMode(redLEDPin, OUTPUT);   // Set red LED pin as output
  pinMode(buzzerPin, OUTPUT);   // Set buzzer pin as output
  myServo.attach(33);           // Attaches the servo on pin 33

  SPI.begin();             // Initialize SPI bus
  mfrc522.PCD_Init();      // Initialize MFRC522
  pinMode(LED_G, OUTPUT);  // Set green LED pin as output
  pinMode(LED_R, OUTPUT);  // Set red LED pin as output
  pinMode(BUZZER, OUTPUT); // Set buzzer pin as output
  noTone(BUZZER);          // Turn off the buzzer initially
  Serial.println("Put your card to the reader..."); // Display message on Serial Monitor
  Serial.println();

  generateOTP(); // Generate the initial OTP
}

void loop() {
  // Look for new cards
  if (mfrc522.PICC_IsNewCardPresent() && mfrc522.PICC_ReadCardSerial()) {
    // Show UID on serial monitor
    Serial.print("UID tag: ");
    String content = "";
    for (byte i = 0; i < mfrc522.uid.size; i++) {
      Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " "); // Print UID bytes
      Serial.print(mfrc522.uid.uidByte[i], HEX);
      content.concat(String(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ")); // Concatenate UID bytes as a string
      content.concat(String(mfrc522.uid.uidByte[i], HEX));
    }
    Serial.println();
    Serial.print("Message: ");
    content.toUpperCase();

    // Check if the RFID card is authorized
    if (content.substring(1) == "96 2C 71 06") { // Change the UID of the authorized card
      Serial.println("Authorized access"); // Display message
      Serial.println();
      delay(500);
      digitalWrite(LED_G, HIGH); // Turn on green LED
      tone(BUZZER, 500); // Sound the buzzer at 500 Hz
      delay(300);
      noTone(BUZZER); // Turn off the buzzer
      myServo.write(180); // Rotate the servo to 180 degrees
      delay(5000); // Wait for 5 seconds
      myServo.write(0); // Return the servo to the initial position (0 degrees)
      digitalWrite(LED_G, LOW); // Turn off green LED
    } else {
      Serial.println("Access denied"); // Display message
      digitalWrite(LED_R, HIGH); // Turn on red LED
      tone(BUZZER, 300); // Sound the buzzer at 300 Hz
      delay(1000);
      digitalWrite(LED_R, LOW); // Turn off red LED
      noTone(BUZZER); // Turn off the buzzer
    }

    // Clear the UID read
    mfrc522.PICC_HaltA();
  }

  // Print the generated OTP to the Serial Monitor
  Serial.println("Generated OTP: " + String(generatedOTP));

  // Wait for user input
  Serial.println("Enter OTP:");
  while (!Serial.available()) {
    // Wait for user input
  }

  // Read user input
  String userOTPString = Serial.readString();
  int userOTP = userOTPString.toInt();

  // Verify OTP
  if (userOTP == generatedOTP) {
    Serial.println("OTP Verified!"); // Display message

    // Rotate the servo to 180 degrees
    myServo.write(180);
    digitalWrite(greenLEDPin, HIGH); // Turn on green LED
    delay(5000); // Wait for 5 seconds

    // Return the servo to the initial position (e.g., 0 degrees)
    myServo.write(0);

    // Blink green LED
    digitalWrite(greenLEDPin, LOW); // Turn off green LED
    digitalWrite(redLEDPin, HIGH); // Turn on red LED

    // Generate a new OTP
    generateOTP();
  } else {
    Serial.println("OTP Verification Failed. Try Again."); // Display message

    // Beep the buzzer when access is denied
    digitalWrite(greenLEDPin, LOW); // Turn off green LED
    delay(500);
    digitalWrite(redLEDPin, HIGH); // Turn on red LED
    digitalWrite(buzzerPin, HIGH); // Turn on the buzzer
    delay(500);
    digitalWrite(buzzerPin, LOW); // Turn off the buzzer
    digitalWrite(redLEDPin, LOW); // Turn off red LED
    tries++;

    // Beep for a longer duration when access is denied after 3 tries
    if (tries == 3) {
      Serial.println("OTP invalid"); // Display message
      digitalWrite(redLEDPin, HIGH); // Turn on red LED
      digitalWrite(buzzerPin, HIGH); // Turn on the buzzer for a longer duration
      delay(1000);
      digitalWrite(buzzerPin, LOW); // Turn off the buzzer
      digitalWrite(redLEDPin, LOW); // Turn off red LED
    }
  }
}

void generateOTP() {
  // Generate a new 6-digit OTP
  generatedOTP = 0;
  for (int i = 0; i < 6; i++) {
    generatedOTP = generatedOTP * 10 + random(0, 10); // Generate a random digit between 0 and 9
  }
}
