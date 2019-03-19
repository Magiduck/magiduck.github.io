---
title: "Arduino sensor basics project"
date: 2019-03-19
tags: [sensors, sensor system engineering, arduino]
header:
  image: "/assets/images/ArduinoSensor/arduino_logo.png"
excerpt: "Sensor basics project for Sensor System Engineering"
---
In the minor Sensor System Engineering I followed the elective "Sensor Basics". In this course we got the assignment to write an application for the arduino Uno with two sensors.

We chose apart from the potentiometer to also use the on-board photosensor of the innoesys educational shield.

{% include video id="gQmgiFgqIwc" provider="youtube" %}

The arduino code can be found below:

```java
/*
 * Final assignment made by Magor Katay and Tijs van Lieshout
 * 
 */
// Servo library
#include <Servo.h>
// Pins for the Innoesys shield on-board LEDs + variable for LED values
const int redPin = 6;
const int greenPin = 9;
const int bluePin = 10;
int LEDValue = 0;
// Pin for the Innoesys shield on-board for the Potentiometer
const int potentioMeter = A1;
int potentioValue = 0;
// Pin for the Innoesys shield on-board for the Photosensor
const int photoSensor = A2;
int photoValue = 0;
// Servo object from the servo library with value
Servo myservo;
int servoValue = 0;
// boolean to allow servo to be active (for the "S" command)
boolean allowServo = true;
// Used to switch between sensors
const int buttonRight = 7; 
int buttonRightState;
int previousRight = 0;
boolean isPotentioOn = true;
// Used to turn on/off the LED display
const int buttonLeft = 8;
int buttonLeftState;
int previousLeft = 0;
boolean isDisplayOn = true;
// Used for blinking led at min and max values
boolean isBlinkOn = false;
unsigned long previousMillis2 = 0;
const long interval2 = 500;
// Used for both buttons
long time = 0;
long debounce = 200;
// Used for sending data every 1Hz with the "C" command
unsigned long previousMillis = 0;
const long interval = 1000;
boolean keepReturningValues = false;
// Pin of the buzzer for the "B" command (also used for the right button)
int buzzer = 5;


void setup() {
  // Outputs
  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);
  pinMode(buzzer, OUTPUT);
  myservo.attach(3);
  // Inputs
  pinMode(potentioMeter, INPUT);
  pinMode(photoSensor, INPUT);
  pinMode(buttonRight, INPUT);
  pinMode(buttonLeft, INPUT);
  // Baud rate for the serial monitor
  Serial.begin(9600);
}

void loop() {
  // For switching between Potentiometer and Photosensor with the right button on the shield
  buttonRightState = digitalRead(buttonRight);
  // Determine if button has been pressed and if <isPotentioOn> should be true (use potentiometer) or false (use photosensor)
  determineButtonRightPress(buttonRightState);
  // Set the value of the LEDs based on the measured sensor value
  setLEDValueBasedOnActiveSensor();
  // For moving the servo (servo won't be allowed if the user toggled it off with a "S" command)
  if(allowServo){
    myservo.write(servoValue);
    delay(15);
  }
  // For turning the LED display on/off again
  buttonLeftState = digitalRead(buttonLeft);
  // For switching with the left button
  determineButtonLeftPress(buttonLeftState);
  // For handling all the commands the user can send to the arduino
  handleCommands(potentioMeter, photoSensor);
  // For turning leds on/off, triggered with the left button or the "L" command
  if(isDisplayOn) {
    displayValuesToLED(LEDValue);  
  } else {
    turnOffLEDs();
  }
  // If the "C" command has been send then keep sending values at a speed of 1hz
  if(keepReturningValues) {
    returnValuesContiniously();
  }
}

// For determining if the right button was pressed and if the potentiometer should be read or the photosensor.
void determineButtonRightPress(int buttonRightState) {
  if(buttonRightState == 1 && previousRight == 0 && millis() - time > debounce) {
    digitalWrite(buzzer, 1);
    delay(50);
    digitalWrite(buzzer, 0);
    if(isPotentioOn) {
      isPotentioOn = false;    
    } else {
      isPotentioOn = true;
    }
    time = millis();
  }
  previousRight = buttonRightState;
}

// Checking which sensor should be measured and converting that value to a value for the LEDs
void setLEDValueBasedOnActiveSensor() {
  // for checking which sensor should be measured
  if(isPotentioOn) {
    potentioValue = analogRead(potentioMeter);
    // mapping sensor value appropiately for the LED and servo
    LEDValue = map(potentioValue, 0, 1023, 0, 255); 
    servoValue = map(potentioValue, 0, 1023, 180, 0);
  } else {
    photoValue = analogRead(photoSensor);
    // mapping sensor value appropiatley for the LED and servo
    LEDValue = map(photoValue, 0, 1023, 0, 255);
    servoValue = map(photoValue, 0, 1023, 180, 0);
  }
}

// For determining if the left button was pressed and if the LED display should be on or not
void determineButtonLeftPress(int buttonLeftState) {
  if(buttonLeftState == 1 && previousLeft == 0 && millis() - time > debounce) {
    if(isDisplayOn) {
      isDisplayOn = false;    
    } else {
      isDisplayOn = true;
    }
    time = millis();
  }
  previousLeft = buttonLeftState;
}

// For handling commands from the serial monitor:
// L  turning on/off LED display
// M  Sending both sensor values once
// C  Sending both sensor values at a speed of 1hz until toggling it off again
// B  Play a note through the buzzer based on the LEDvalue
// S  Toggling the use of the servo as a display
void handleCommands(int potentioMeter, int photoSensor) {
  int command;
  if (Serial.available() > 0) {
    command = Serial.read();
    if(command == 'L') {
      if (isDisplayOn) {
        isDisplayOn = false;  
      } else {
        isDisplayOn = true;
      }
    } else if(command == 'M') {
        potentioValue = analogRead(potentioMeter);
        photoValue = analogRead(photoSensor);
        Serial.println("Potentiometer value: " + String(potentioValue) + " Photometer value: " + String(photoValue));
    } else if(command == 'C') {
        if(keepReturningValues) {
          keepReturningValues = false;
        } else {
          keepReturningValues = true;
        }
    } else if(command == 'B') {
        if(LEDValue >= 0 && LEDValue <= 85) {
          tone(buzzer, 31);
          delay(50);
          noTone(buzzer);
        }
        else if(LEDValue > 85 && LEDValue <= 170){
          tone(buzzer, 494);
          delay(50);
          noTone(buzzer);
        }
        else if(LEDValue > 170 && LEDValue <= 255){
          tone(buzzer, 3951);
          delay(50);
          noTone(buzzer);
        }
    } else if(command == 'S') {
      if(allowServo){
        allowServo = false;
        Serial.println("You turned the Servo off");
      } else {
          allowServo = true;
          Serial.println("You turned the Servo on");
      }
    }
  }
}

// Displaying values between 0 and 255 to three LEDs (digitally, it was at some point analog but the servo library causes problems)
void displayValuesToLED(int value) {
  if(value <= 0) {
    digitalWrite(greenPin, 0);
    digitalWrite(bluePin, 0);
    // For blinking at min and max value
    unsigned long currentMillis2 = millis();
    if (currentMillis2 - previousMillis2 >= interval2) {
      previousMillis2 = currentMillis2;
      if(isBlinkOn){
        digitalWrite(redPin, 0);
        isBlinkOn = false;
      } else {
        digitalWrite(redPin, 1);
        isBlinkOn = true;
      }
    }
  }
  else if(value > 0 && value <= 85) {
    digitalWrite(greenPin, 0);
    digitalWrite(bluePin, 0);
    digitalWrite(redPin, 1);
  }
  else if(value > 85 && value <= 170){
    digitalWrite(redPin, 0);
    digitalWrite(bluePin, 0);
    digitalWrite(greenPin, 1);
  }
  else if(value > 170 && value < 255){
    digitalWrite(greenPin, 0);
    digitalWrite(redPin, 0);
    digitalWrite(bluePin, 1);
  }
  else if(value >= 255) {
    digitalWrite(greenPin, 0);
    digitalWrite(redPin, 0);
    // For blinking at min and max value
    unsigned long currentMillis2 = millis();
    if (currentMillis2 - previousMillis2 >= interval2) {
      previousMillis2 = currentMillis2;
      if(isBlinkOn){
        digitalWrite(bluePin, 0);
        isBlinkOn = false;
      } else {
        digitalWrite(bluePin, 1);
        isBlinkOn = true;
      }
    }
  }
}

// Turning all LEDs off at the same time, can be triggered through the "L" command or through the left button press on the shield
void turnOffLEDs() {
  digitalWrite(greenPin, 0);
  digitalWrite(bluePin, 0);
  digitalWrite(redPin, 0);  
}

// Used for sending both values at 1hz if the user has send the "U" command
void returnValuesContiniously() {
  unsigned long currentMillis = millis();
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;
    potentioValue = analogRead(potentioMeter);
    photoValue = analogRead(photoSensor);
    Serial.println("Potentiometer value: " + String(potentioValue) + " Photosensor value: " + String(photoValue));
  }
}
```