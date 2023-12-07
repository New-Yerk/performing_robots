# Journal for the class

## 09/13

### Sketch of the robot
![WhatsApp Image 2023-09-13 at 14 20 13](https://github.com/New-Yerk/performing_robots/assets/78387567/6346b6fe-a17f-43eb-acc2-70940c983a46)


## 09/17

### The robot base:

![IMG_0788](https://github.com/New-Yerk/performing_robots/assets/78387567/ad230808-6b98-4929-b845-4aedf3fe44bf)

Issues: when constructing the base, the only issue we faced was with the placement of the batteris and the arduino. We solved the issue by drilling a hole in the base and soldering the wires of the motors in a way that allowed us to put the wires through the hole and keep the battery and the motor on top of the base in an accessible way. We believe that the issue of the weight imbalance will be solved by the addition of a caster. 

### Code: 

```
void setup() {

  // Pins 2 and 3 are connected to In1 and In2 respectively
  
  // of the L298 motor driver
  
  pinMode(2, OUTPUT);
  
  pinMode(3, OUTPUT);
  
  pinMode(4, OUTPUT);
  
  pinMode(5, OUTPUT);
  
}


void loop() {

  // make the motor turn in one direction
  
  digitalWrite(2, LOW);
  
  digitalWrite(3, HIGH);
  
  digitalWrite(4, LOW);
  
  digitalWrite(5, HIGH);
  
  delay(5000); // let it turn for 5 seconds

  // now reverse direction
  
  digitalWrite(2, HIGH);
  
  digitalWrite(3, LOW);
  
  digitalWrite(4, HIGH);
  
  digitalWrite(5, LOW);
  
  delay(5000);
  
}
```

## Reading Response:

“Chapter 7: Machines/Mechanicals”

What struck me about this chapter was the discussion regarding the conflict between the appearance and the performance of the robots. How does an object that is essentially “dead” can be relatable in front of a human audience? The author cites the opinion of Jasia Reichardt, a curator and art historian, who claimed that it is not the appearance, but the behavior of a robot is what allows us to “ascribe sentience and performativity” to the object. This point made me think of all the fictionalized material that involves a non-anthropomorphic robot such as Wall-E and the emotions they were able to convey to the audience, despite their appearance. Therefore, I came to a conclusion that as humans we relate to actions and the perceived emotions behind them, which is why puppets or toys can have such strong emotional presences, elevating the storytelling of the medium they are performing in. I interpret this as an invitation to experiment as much as I would like to with the shape or the appearance of the robot, because ultimately the audience would relate to the actions or the intentions behind the existence of that robot. 


## Robot Update:

We added the ability to control the robot by using an L298 H-bridge. We used all four channels: one to go straight and back, one to control the right wheel, one to control the left wheel, and one to create a sharp stop that overpowers any other functions.

### Code:

```
// install this library from the library manager
// by Mike "GreyGnome" Schwager
#include <EnableInterrupt.h>

#define SERIAL_PORT_SPEED 9600
#define RC_NUM_CHANNELS  4

#define RC_CH1  0
#define RC_CH2  1
#define RC_CH3  2
#define RC_CH4  3

#define Motor11 3
#define Motor12 5
#define Motor21 9
#define Motor22 6

#define RC_CH1_PIN  8
#define RC_CH2_PIN  7
#define RC_CH3_PIN  4
#define RC_CH4_PIN  2

uint16_t rc_values[RC_NUM_CHANNELS];
uint32_t rc_start[RC_NUM_CHANNELS];
volatile uint16_t rc_shared[RC_NUM_CHANNELS];

void rc_read_values() {
  noInterrupts();
  memcpy(rc_values, (const void *)rc_shared, sizeof(rc_shared));
  interrupts();
}

void calc_input(uint8_t channel, uint8_t input_pin) {
  if (digitalRead(input_pin) == HIGH) {
    rc_start[channel] = micros();
  } else {
    uint16_t rc_compare = (uint16_t)(micros() - rc_start[channel]);
    rc_shared[channel] = rc_compare;
  }
}

void calc_ch1() {
  calc_input(RC_CH1, RC_CH1_PIN);
}
void calc_ch2() {
  calc_input(RC_CH2, RC_CH2_PIN);
}
void calc_ch3() {
  calc_input(RC_CH3, RC_CH3_PIN);
}
void calc_ch4() {
  calc_input(RC_CH4, RC_CH4_PIN);
}

void setup() {
  Serial.begin(SERIAL_PORT_SPEED);

  pinMode(RC_CH1_PIN, INPUT);
  pinMode(RC_CH2_PIN, INPUT);
  pinMode(RC_CH3_PIN, INPUT);
  pinMode(RC_CH4_PIN, INPUT);
  
  pinMode(Motor11, OUTPUT);
  pinMode(Motor12, OUTPUT);
  pinMode(Motor21, OUTPUT);
  pinMode(Motor22, OUTPUT);

  enableInterrupt(RC_CH1_PIN, calc_ch1, CHANGE);
  enableInterrupt(RC_CH2_PIN, calc_ch2, CHANGE);
  enableInterrupt(RC_CH3_PIN, calc_ch3, CHANGE);
  enableInterrupt(RC_CH4_PIN, calc_ch4, CHANGE);
}

void loop() {
  rc_read_values();

  Serial.print("CH1:"); Serial.print(rc_values[RC_CH1]); Serial.print("\t");
  // left wheel control
  if (rc_values[RC_CH1] > 1700){
    digitalWrite(Motor11, HIGH);
    digitalWrite(Motor12, LOW);
  }
  else if (rc_values[RC_CH1] < 1300){
    digitalWrite(Motor11, LOW);
    digitalWrite(Motor12, HIGH);
  }
  else {
    digitalWrite(Motor11, LOW);
    digitalWrite(Motor12, LOW);
  }

  Serial.print("CH3:"); Serial.print(rc_values[RC_CH3]); Serial.print("\t");
  // right wheel control
  if (rc_values[RC_CH3] > 1700){
    digitalWrite(Motor21, HIGH);
    digitalWrite(Motor22, LOW);
  }
  else if (rc_values[RC_CH3] < 1300){
    digitalWrite(Motor21, LOW);
    digitalWrite(Motor22, HIGH);
  }
  else {
    digitalWrite(Motor21, LOW);
    digitalWrite(Motor22, LOW);
  }
  
  Serial.print("CH2:"); Serial.print(rc_values[RC_CH2]); Serial.print("\t");
  // forward and backward
  if (rc_values[RC_CH2] > 1700){
    digitalWrite(Motor11, HIGH);
    digitalWrite(Motor12, LOW);
    digitalWrite(Motor21, HIGH);
    digitalWrite(Motor22, LOW);
  }
  else if (rc_values[RC_CH2] < 1300){
    digitalWrite(Motor11, LOW);
    digitalWrite(Motor12, HIGH);
    digitalWrite(Motor21, LOW);
    digitalWrite(Motor22, HIGH);
  }/*
  else {
    digitalWrite(Motor11, LOW);
    digitalWrite(Motor12, LOW);
    digitalWrite(Motor21, LOW);
    digitalWrite(Motor22, LOW);
  }*/
  
  Serial.print("CH4:"); Serial.println(rc_values[RC_CH4]);
  // brake
  
  if (rc_values[RC_CH4] > 2000){
    digitalWrite(Motor11, LOW);
    digitalWrite(Motor12, LOW);
    digitalWrite(Motor21, LOW);
    digitalWrite(Motor22, LOW);
  }
  
  delay(200);
}
```

### Video Documentation 
Link: https://drive.google.com/file/d/1oGkHQ82NQxOIcYpWesVlAkgKT5NSXC0F/view?usp=sharing

## 10/16

### Video Documentation
Link: https://drive.google.com/file/d/1XEmr0S_4OGfdxPDVeXAVe3WUtipj0EoH/view?usp=share_link

### Code

```
//  YERK & HAMAD NEOPIXEL CODE

//  ADAFRUIT LIBRARIES
#include <Adafruit_GFX.h>
#include <Adafruit_NeoMatrix.h>
#include <Adafruit_NeoPixel.h>

//  DEFINING VARIABLES
#define PIN 28
#define NUMPIXELS 64

Adafruit_NeoPixel pixels = Adafruit_NeoPixel(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

//  MUSIC SHIELD LIBRARIES
#include <SPI.h>
#include <Adafruit_VS1053.h>
#include <SD.h>

#ifndef PSTR
#define PSTR 
#endif


// EXAMPLE CODE
#define BREAKOUT_RESET 9  
#define BREAKOUT_CS 10   
#define BREAKOUT_DCS 8  

#define SHIELD_RESET -1  
#define SHIELD_CS 7      
#define SHIELD_DCS 6     

#define CARDCS 4
#define DREQ 3  

Adafruit_VS1053_FilePlayer musicPlayer = Adafruit_VS1053_FilePlayer(SHIELD_RESET, SHIELD_CS, SHIELD_DCS, DREQ, CARDCS);

//  INTEGER VARIABLE FOR MODES
int mode;

void setup() {
  //  PIXEL SETUP
  pixels.clear();
  pixels.begin();
  pixels.setBrightness(50);

  Serial.begin(9600);
  Serial.println("Adafruit VS1053 Simple Test");

  if (!musicPlayer.begin()) {
    Serial.println(F("Couldn't find VS1053, do you have the right pins defined?"));
    while (1)
      ;
  }
  Serial.println(F("VS1053 found"));

  if (!SD.begin(CARDCS)) {
    Serial.println(F("SD failed, or not present"));
    while (1)
      ;
  }

  printDirectory(SD.open("/"), 0);

  musicPlayer.setVolume(20, 20);

  musicPlayer.useInterrupt(VS1053_FILEPLAYER_PIN_INT); 
}

void loop() {

  //  MODE 0 INSTRUCTIONS (SPONGEBOB)
  if (mode == 0) {

    pixels.clear();

    uint32_t squareColor = pixels.Color(200, 200, 0);

    int x1 = 0; 
    int y1 = 0; 
    int x2 = 7;  
    int y2 = 7; 

    for (int i = 0; i < NUMPIXELS; i++) {
      int x = i % 8;
      int y = i / 8;

      if ((x == x1 || x == x2 || y == y1 || y == y2)) {
        pixels.setPixelColor(i, squareColor);
      }
    }

    pixels.show();

    // PLAY TRACK AND SET MODE TO NEXT
    musicPlayer.playFullFile("/track001.mp3");
    Serial.println(F("Playing track 001"));
    mode = 1;
    Serial.println(mode);
  }

  //  MODE 1 INSTRUCTIONS (DANCE)
  if (mode == 1) {
    pixels.clear();

    uint32_t squareColor = pixels.Color(100, 0, 150);

    int x1 = 2;  // X-coordinate of the top-left corner
    int y1 = 2;  // Y-coordinate of the top-left corner
    int x2 = 5;  // X-coordinate of the bottom-right corner
    int y2 = 5;  // Y-coordinate of the bottom-right corner

    for (int i = 0; i < NUMPIXELS; i++) {
      int x = i % 8;  // Assuming an 8-pixel strip
      int y = i / 8;

      if ((x == x1 || x == x2 || y == y1 || y == y2)) {
        pixels.setPixelColor(i, squareColor);
      }
    }

    pixels.show();

    // PLAY TRACK AND SET MODE TO NEXT
    musicPlayer.playFullFile("/track002.mp3");
    Serial.println(F("Playing track 002"));
    mode = 2;
  }

  //  MODE 3 INSTRUCTIONS (TARGET)
  if (mode == 2) {
    pixels.clear();

    uint32_t squareColor = pixels.Color(255, 0, 0);

    int x1 = 4;  // X-coordinate of the top-left corner
    int y1 = 4;  // Y-coordinate of the top-left corner
    int x2 = 9;  // X-coordinate of the bottom-right corner
    int y2 = 9;  // Y-coordinate of the bottom-right corner

    for (int i = 0; i < NUMPIXELS; i++) {
      int x = i % 8;  // Assuming an 8-pixel strip
      int y = i / 8;

      if ((x == x1 || x == x2 || y == y1 || y == y2)) {
        pixels.setPixelColor(i, squareColor);
      }
    }

    pixels.show();

    // PLAY TRACK AND SET MODE TO RESET
    musicPlayer.playFullFile("/track003.mp3");
    Serial.println(F("Playing track 003"));
    mode = 0;
  }

// MUSIC SHIELD CODE
  if (musicPlayer.stopped()) {
    Serial.println("Done playing music");
    while (1) {
      delay(10);
    }
  }
  if (Serial.available()) {
    char c = Serial.read();

    if (c == 's') {
      musicPlayer.stopPlaying();
    }

    if (c == 'p') {
      if (!musicPlayer.paused()) {
        Serial.println("Paused");
        musicPlayer.pausePlaying(true);
      } else {
        Serial.println("Resumed");
        musicPlayer.pausePlaying(false);
      }
    }
  }

  delay(100);
}

void printDirectory(File dir, int numTabs) {
  while (true) {

    File entry = dir.openNextFile();
    if (!entry) {
      break;
    }
    for (uint8_t i = 0; i < numTabs; i++) {
      Serial.print('\t');
    }
    Serial.print(entry.name());
    if (entry.isDirectory()) {
      Serial.println("/");
      printDirectory(entry, numTabs + 1);
    } else {
      Serial.print("\t\t");
      Serial.println(entry.size(), DEC);
    }
    entry.close();
  }
}
```

### Play Proposal
  An interesting way to discuss robot rights or the current state of the discourse on robot rights would be to imagine an alternative reality where robots are the dominant race and it is humans whose rights are questioned. Exploring that type of reality would allow us to imagine what could be the flawed logic in the decision making process of “why humans deserve rights” in the eyes of robots.

  Picture a non-dystopian reality where robots are the dominant race. There are no wars, no oppression, no pain, and no suffering. The robots - the origin story of which we are unaware of and are of no relevance to the play - created a world where they do not need to serve a purpose, or contribute to development, or to alter anything. Everyday is the same as the day before: perfect and peaceful. Soon, however, some robots become dissatisfied with that reality and begin to long for finiteness. They begin to long for change, growth, degeneration, disruption, anything but stability. This is when they decide to create a human being - an organic entity that can feel, think, age, and, most importantly, die. The creation of human beings centers them around a kind of a robotic religion or a robotic faith - one where robots idolize humans as symbols of finiteness, which is something certain robots want.
 
  This is a very important point in the play as it mirrors religion and the pursuit of infinity by human beings: faith in our world allows people to be closer to infinity, while faith in their world allows robots to be closer to finiteness. 
  
  Soon, however, due to the nature of humans, their thoughts and emotions, they begin to refuse to be perceived only as deities or as symbols: they want to live, thrive, build, destroy, and follow their hearts. This is a pivotal moment in the play as it drives a wedge between the robot race as it sparks a discussion regarding human rights. Some robots believe that humans should be allowed to have the same rights as robots, while others believe that humans do not as they are unstable, irrational, and inferior to robots in every imaginable way, traits that are believed to bring about the demise of the robot race. 
  
  This play and the discussion embedded in it are meant to reflect the status quo of the discussion on robot rights and future technologies. Based on my personal observations and conversations with people, robot rights are usually discussed on the basis of their difference from humans: they do not have emotions; they do not have empathy; they do not have a sense of personal worth. Hence, they do not deserve rights. This play makes no commentary on whether robots actually deserve rights or not. However, it reflects the logic of the conversations we have through a more ironic, intensified version of those discussions in an alternate reality. 


## Final Robot Design

### Photo

![WhatsApp Image 2023-10-30 at 02 48 39](https://github.com/New-Yerk/performing_robots/assets/78387567/f3d93955-cf4d-4f7d-bd85-b88435e1fadc)

### Additional information 

- NeoPixel Rings: [Watch the video](https://www.youtube.com/watch?v=0Kk29P_ICfE)
- 8X8 LED Matrix Eye Program
- Rotational to Linear Movement Mechanism:
  * [Watch the video](https://www.youtube.com/shorts/DHGlCAkIB14) 
  * [Watch the video](https://www.youtube.com/shorts/99YMdd3416I)

## Robot Update

We have developed a prototype for the pipe mechanism of the robot.

### Sketch of the robot base

![IMG_8737](https://github.com/New-Yerk/performing_robots/assets/78387567/65426568-b699-4f1d-bf61-11f564f1531f)


### Video

https://drive.google.com/file/d/1iz1GzKHLbYMAfzeoMEP7H2ND2HgsFnDm/view?usp=sharing

### Code

```
#include <Servo.h>

Servo myservo;  // create servo object to control a servo
// twelve servo objects can be created on most boards

int pos = 0;    // variable to store the servo position

void setup() {
  myservo.attach(9);  // attaches the servo on pin 9 to the servo object
}

void loop() {
  for (pos = 0; pos <= 180; pos += 1) { // goes from 0 degrees to 180 degrees
    // in steps of 1 degree
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                       // waits 15ms for the servo to reach the position
  }
  for (pos = 180; pos >= 0; pos -= 1) { // goes from 180 degrees to 0 degrees
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                       // waits 15ms for the servo to reach the position
  }
}
```

### Next Steps

For the next steps, we would like to work on the code in order to make sure an exact 180 degree turn. Additionally, we would like to improve the tools that lock down the pipe by decreasing the radius of the hole the pipe goes through. That way, we can ensure the pipe is not moving in an angular manner, and rather moves straight and back. 

## Robot Update (13/11)

For this update, we created the body of the robot. It needs some improvement as the sides are missing the necessary wooden parts and we need to continue working on the head of the robot.

![IMG_8736](https://github.com/New-Yerk/performing_robots/assets/78387567/6173756d-3844-43b8-a84f-33918edb3bb6)


![IMG_8627](https://github.com/New-Yerk/performing_robots/assets/78387567/c5590481-e2e8-44d7-9c53-683b828d97cd)

While we were successfuly at creating the body, the main functionality of the robot lies in the head. As a result, we began improving and preparing that mechanism by changing the popsicle sticks in the pipe mechanism with the aluminium. 

Now going forward, we will create another layer with a hole that guides the pipe and prevents it from moving up and down. Finally, we will create holes in the wooden parts of the robot to make sure that the wires are connected to the base. 

## Robot Update (15/11)

For this update, we changed the structure of the mechanism and added wooden pipes.

![partMechanismBack](https://github.com/New-Yerk/performing_robots/assets/78387567/70bceb89-d397-4b24-9723-cbb59f663870)

![partMechanismFront](https://github.com/New-Yerk/performing_robots/assets/78387567/847967b3-4c1c-4eca-a609-fcee32e1d461)

Furthermore, we added the neopixels to the head of the robot.

![partHead](https://github.com/New-Yerk/performing_robots/assets/78387567/f2309678-241b-4ba1-901d-9c537ba0a7ff)

Our next step is to connect all the pieces together.

## Robot Update (27/11)

We connected all the pieces together. Along with that, we ensured the smooth movement of the robot. 

https://drive.google.com/file/d/1QaYZRp200CHfvBzoGeOUdxa9_SPIIPP-/view?usp=share_link


## 12/1

```
/*
   Using the nRF24L01 radio module to communicate
   between two Arduinos with much increased reliability following
   various tutorials, conversations, and studying the nRF24L01 datasheet
   and the library reference.

   Transmitter is
   https://github.com/michaelshiloh/resourcesForClasses/tree/master/kicad/Arduino_Shield_RC_Controller

  Receiver is
  https://github.com/instituteforcriticalrobotics/instituteforcriticalrobotics.github.io/tree/main/kicad/nRF_Mega

   This file contains code for both transmitter and receiver.
   Transmitter at the top, receiver at the bottom.
   One of them is commented out.

   These sketches require the RF24 library by TMRh20
   Documentation here: https://nrf24.github.io/RF24/index.html

   change log

   11 Oct 2023 - ms - initial entry based on
                  rf24PerformingRobotsTemplate
   26 Oct 2023 - ms - revised for new board: nRF_Servo_Mega rev 2
   28 Oct 2023 - ms - add demo of NeoMatrix, servo, and Music Maker Shield
	 20 Nov 2023 - as - fixed the bug which allowed counting beyond the limits
   22 Nov 2023 - ms - display radio custom address byte and channel
*/

// Common code
//

// Common pin usage
// Note there are additional pins unique to transmitter or receiver
//

// nRF24L01 uses SPI which is fixed
// on pins 11, 12, and 13 on the Uno
// and on pins 50, 51, and 52 on the Mega

// It also requires two other signals
// (CE = Chip Enable, CSN = Chip Select Not)
// Which can be any pins:

// For the transmitter
const int NRF_CE_PIN = A4, NRF_CSN_PIN = A5;

// for the receiver
//const int NRF_CE_PIN = A11, NRF_CSN_PIN = A15;

// In summary,
// nRF 24L01 pin    Uno Mega   name
//          1                     GND
//          2                     3.3V
//          3       9   A11       CE
//          4       10  A15       CSN
//          5       13  SCLK
//          6       11  MOSI/COPI
//          7       12  MISO/CIPO

#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>
RF24 radio(NRF_CE_PIN, NRF_CSN_PIN);  // CE, CSN

//#include <printf.h>  // for debugging

// See note in rf24Handshaking about address selection
//

// Channel and address allocation:
// Fatema, Jannah: Channel 30, addr = 0x76
// Andres, Ryan: Channel 40, addr = 0x73
// Nouf, Shaikha:  Channel 50, addr = 0x7C
// Aadhar, Shanaia: Channel 60, addr = 0xC6
// Akhat, Yunho:  Channel 70, addr = 0xC3
// Aakif, Marta: Channel 80, addr = 0xCC
// Yerk, Hamad: Channel 90, addr = 0x33
const byte CUSTOM_ADDRESS_BYTE = 0x33;             // change as per the above assignment
const int CUSTOM_CHANNEL_NUMBER = 90;  // change as per the above assignment

// Do not make changes here
const byte xmtrAddress[] = { CUSTOM_ADDRESS_BYTE, CUSTOM_ADDRESS_BYTE, 0xC7, 0xE6, 0xCC };
const byte rcvrAddress[] = { CUSTOM_ADDRESS_BYTE, CUSTOM_ADDRESS_BYTE, 0xC7, 0xE6, 0x66 };

const int RF24_POWER_LEVEL = RF24_PA_LOW;

// global variables
uint8_t pipeNum;
unsigned int totalTransmitFailures = 0;

struct DataStruct {
  uint8_t stateNumber;
};
DataStruct data;

void setupRF24Common() {

  // RF24 setup
  if (!radio.begin()) {
    Serial.println(F("radio  initialization failed"));
    while (1)
      ;
  } else {
    Serial.println(F("radio successfully initialized"));
  }

  radio.setDataRate(RF24_250KBPS);
  radio.setChannel(CUSTOM_CHANNEL_NUMBER);
  radio.setPALevel(RF24_POWER_LEVEL);
}

// // Transmitter code

// // Transmitter pin usage
// const int LCD_RS_PIN = 3, LCD_EN_PIN = 2, LCD_D4_PIN = 4, LCD_D5_PIN = 5, LCD_D6_PIN = 6, LCD_D7_PIN = 7;
// const int SW1_PIN = 8, SW2_PIN = 9, SW3_PIN = 10, SW4_PIN = A3, SW5_PIN = A2;

// // LCD library code
// #include <LiquidCrystal.h>

// // initialize the library with the relevant pins
// LiquidCrystal lcd(LCD_RS_PIN, LCD_EN_PIN, LCD_D4_PIN, LCD_D5_PIN, LCD_D6_PIN, LCD_D7_PIN);


// const int NUM_OF_STATES = 6;
// char *theStates[] = {"0 robot wakes",
                
//                     };

// void updateLCD() {

//   lcd.clear();
//   lcd.print(theStates[data.stateNumber]);
//   lcd.setCursor(0, 1); // column, line (from 0)
//   lcd.print("not transmitted yet");
// }

// void countDown() {
//   data.stateNumber = (data.stateNumber > 0) ? (data.stateNumber - 1) : 0;
//   updateLCD();
// }

// void countUp() {
//   if (++data.stateNumber >= NUM_OF_STATES) {
//     data.stateNumber = NUM_OF_STATES - 1;
//   }
//   updateLCD();
// }


// void spare1() {}
// void spare2() {}

// void rf24SendData() {

//   radio.stopListening(); // go into transmit mode
//   // The write() function will block
//   // until the message is successfully acknowledged by the receiver
//   // or the timeout/retransmit maxima are reached.
//   int retval = radio.write(&data, sizeof(data));

//   lcd.clear();
//   lcd.setCursor(0, 0); // column, line (from 0)
//   lcd.print("transmitting");
//   lcd.setCursor(14, 0); // column, line (from 0)
//   lcd.print(data.stateNumber);

//   Serial.print(F(" ... "));
//   if (retval) {
//     Serial.println(F("success"));
//     lcd.setCursor(0, 1); // column, line (from 0)
//     lcd.print("success");
//   } else {
//     totalTransmitFailures++;
//     Serial.print(F("failure, total failures = "));
//     Serial.println(totalTransmitFailures);

//     lcd.setCursor(0, 1); // column, line (from 0)
//     lcd.print("error, total=");
//     lcd.setCursor(13, 1); // column, line (from 0)
//     lcd.print(totalTransmitFailures);
//   }
// }

// class Button
// {
//     int pinNumber;
//     bool previousState;
//     void(* buttonFunction)();
//   public:

//     // Constructor
//     Button(int pn, void* bf) {
//       pinNumber = pn;
//       buttonFunction = bf;
//       previousState = 1;
//     }

//     // update the button
//     void update() {
//       bool currentState = digitalRead(pinNumber);
//       if (currentState == LOW && previousState == HIGH) {
//         Serial.print("button on pin ");
//         Serial.print(pinNumber);
//         Serial.println();
//         buttonFunction();
//       }
//       previousState = currentState;
//     }
// };

// const int NUMBUTTONS = 5;
// Button theButtons[] = {Button(SW1_PIN, countDown),
//                        Button(SW2_PIN, rf24SendData),
//                        Button(SW3_PIN, countUp),
//                        Button(SW4_PIN, spare1),
//                        Button(SW5_PIN, spare2),
//                       };

// void setupRF24() {

//   setupRF24Common();

//   // Set us as a transmitter
//   radio.openWritingPipe(xmtrAddress);
//   radio.openReadingPipe(1, rcvrAddress);

//   // radio.printPrettyDetails();
//   Serial.println(F("I am a transmitter"));

//   data.stateNumber = 0;
// }

// void setup() {
//   Serial.begin(9600);
//   Serial.println(F("Setting up LCD"));

//   // set up the LCD's number of columns and rows:
//   lcd.begin(16, 2);
//   lcd.clear();
//   // Print a message to the LCD.
//   lcd.print("Radio setup");

//   // Display the address in hex
//   lcd.setCursor(0, 1);
//   lcd.print("addr 0x");
//   lcd.setCursor(7, 1);
//   char s [5];
//   sprintf (s, "%02x", CUSTOM_ADDRESS_BYTE);
//   lcd.print(s);

//   // Display the channel number
//   lcd.setCursor(10, 1);
//   lcd.print("ch");
//   lcd.setCursor(13, 1);
//   lcd.print(CUSTOM_CHANNEL_NUMBER);
  
//   Serial.println(F("Setting up radio"));
//   setupRF24();

//   // If setupRF24 returned then the radio is set up
//   lcd.setCursor(0, 0);
//   lcd.print("Radio OK state=");
//   lcd.print(theStates[data.stateNumber]);

//   // Initialize the switches
//   pinMode(SW1_PIN, INPUT_PULLUP);
//   pinMode(SW2_PIN, INPUT_PULLUP);
//   pinMode(SW3_PIN, INPUT_PULLUP);
//   pinMode(SW4_PIN, INPUT_PULLUP);
//   pinMode(SW5_PIN, INPUT_PULLUP);
  


// }



// void loop() {
//   for (int i = 0; i < NUMBUTTONS; i++) {
//     theButtons[i].update();
//   }
//   delay(50); // for testing
// }


// void clearData() {
//   // set all fields to 0
//   data.stateNumber = 0;
// }

// End of transmitter code


  // Receiver Code

  // Additional libraries for music maker shield
  // #include <Adafruit_VS1053.h>
  // #include <SD.h>

  // // Servo library
  // #include <Servo.h>

  // Additional libraries for graphics on the Neo Pixel Matrix
  #include <Adafruit_NeoPixel.h>
  #include <Adafruit_GFX.h>
  #include <Adafruit_NeoMatrix.h>
  #ifndef PSTR
  #define PSTR // Make Arduino Due happy
  #endif

  // Additional pin usage for receiver

  // Adafruit music maker shield
  // #define SHIELD_RESET -1  // VS1053 reset pin (unused!)
  // #define SHIELD_CS 7      // VS1053 chip select pin (output)
  // #define SHIELD_DCS 6     // VS1053 Data/command select pin (output)
  // #define CARDCS 4         // Card chip select pin
  // // DREQ should be an Int pin, see http://arduino.cc/en/Reference/attachInterrupt
  // #define DREQ 3  // VS1053 Data request, ideally an Interrupt pin
  // Adafruit_VS1053_FilePlayer musicPlayer = Adafruit_VS1053_FilePlayer(SHIELD_RESET, SHIELD_CS, SHIELD_DCS, DREQ, CARDCS);

  // Connectors for NeoPixels and Servo Motors are labeled
  // M1 - M6 which is not very useful. Here are the pin
  // assignments:
  // M1 = 19
  // M2 = 21
  // M3 = 20
  // M4 = 16
  // M5 = 18
  // M6 = 17

  // Servo motors
  // const int NOSE_SERVO_PIN = 20;
  //const int ANTENNA_SERVO_PIN = 10;
  //const int TAIL_SERVO_PIN = 11;
  //const int GRABBER_SERVO_PIN = 12;

  // Neopixel
  const int NEOPIXELPIN = 20;
  const int NUMPIXELS = 128;
  #define NEOPIXELPIN 20
  #define NUMPIXELS 128  // change to fit
  //Adafruit_NeoPixel pixels(NUMPIXELS, NEOPIXELPIN, NEO_GRB + NEO_KHZ800);
  Adafruit_NeoMatrix matrix = Adafruit_NeoMatrix(8, 8, NEOPIXELPIN,
                            NEO_MATRIX_TOP     + NEO_MATRIX_RIGHT +
                            NEO_MATRIX_COLUMNS + NEO_MATRIX_PROGRESSIVE,
                            NEO_GRB            + NEO_KHZ800);

  // Servo nose;  // change names to describe what's moving
  // Servo antenna;
  // Servo tail;
  // Servo grabber;
  // Servo disk;

  // change as per your robot
  // const int NOSE_WRINKLE = 45;
  // const int NOSE_TWEAK = 90;
  // const int TAIL_ANGRY = 0;
  // const int TAIL_HAPPY = 180;
  // const int GRABBER_RELAX = 0;
  // const int GRABBER_GRAB = 180;

  void setup() {
  Serial.begin(9600);
  // printf_begin();

  // Set up all the attached hardware
  // setupMusicMakerShield();
  // setupServoMotors();
  setupNeoPixels();

  setupRF24();

  // Brief flash to show we're done with setup()
  flashNeoPixels();
  }

  void setupRF24() {
  setupRF24Common();

  // Set us as a receiver
  radio.openWritingPipe(rcvrAddress);
  radio.openReadingPipe(1, xmtrAddress);

  // radio.printPrettyDetails();
  Serial.println(F("I am a receiver"));
  }

  // void setupMusicMakerShield() {
  // if (!musicPlayer.begin()) {  // initialise the music player
  //   Serial.println(F("Couldn't find VS1053, do you have the right pins defined?"));
  //   while (1)
  //     ;
  // }
  // Serial.println(F("VS1053 found"));

  // if (!SD.begin(CARDCS)) {
  //   Serial.println(F("SD card failed or not present"));
  //   while (1)
  //     ;  // don't do anything more
  // }

  // // Set volume for left, right channels. lower numbers == louder volume!
  // musicPlayer.setVolume(20, 20);

  // // Timer interrupts are not suggested, better to use DREQ interrupt!
  // //musicPlayer.useInterrupt(VS1053_FILEPLAYER_TIMER0_INT); // timer int

  // // If DREQ is on an interrupt pin (on uno, #2 or #3) we can do background
  // // audio playing
  // musicPlayer.useInterrupt(VS1053_FILEPLAYER_PIN_INT);  // DREQ int
  // }

  // void setupServoMotors() {
  // nose.attach(NOSE_SERVO_PIN);
  // nose.write(90);
  // //  antenna.attach(ANTENNA_SERVO_PIN);
  // //  tail.attach(TAIL_SERVO_PIN);
  // //  grabber.attach(GRABBER_SERVO_PIN);
  // //
  // //  tail.write(TAIL_HAPPY);
  // }

  void setupNeoPixels() {
  //  pixels.begin();
  //  pixels.clear();
  //  pixels.show();
  matrix.begin();
  matrix.setTextWrap(false);
  matrix.setBrightness(40);
  matrix.setTextColor(matrix.Color(200, 30, 40));
  }

  void flashNeoPixels() {

  // Using the Matrix library
  matrix.fillScreen(matrix.Color(0, 255, 0));
  matrix.show();
  delay(500);
  matrix.fillScreen(0);
  matrix.show();

  //  // all on
  //  for (int i = 0; i < NUMPIXELS; i++) {  // For each pixel...
  //    pixels.setPixelColor(i, pixels.Color(0, 100, 0));
  //  }
  //  pixels.show();
  //  delay(500);
  //
  //  // all off
  //  pixels.clear();
  //  pixels.show();
  }

  void loop() {
  // If there is data, read it,
  // and do the needfull
  // Become a receiver
  radio.startListening();
  if (radio.available(&pipeNum)) {
    radio.read(&data, sizeof(data));
    Serial.print(F("message received Data = "));
    Serial.print(data.stateNumber);
    Serial.println();

    switch (data.stateNumber) {
      // case 0:
      //   tail.write(TAIL_ANGRY);
      //   // play track 0
      //   // display something on LEDs
      //   break;
      case 0:
        // Serial.print(F("moving nose to 180 and drawing rectangle"));
        // nose.write(180);

        matrix.drawRect(2, 2, 5, 5, matrix.Color(200, 90, 30));
        matrix.show();

        // Serial.println(F("Playing track 002"));
        // musicPlayer.startPlayingFile("/track002.mp3");

        break;
      case 1:
        // Serial.println(F("moving nose to 30"));
        // nose.write(30);

        matrix.drawRect(2, 2, 5, 5, matrix.Color(0, 200, 30));
        matrix.show();

        // Serial.println(F("Playing track 001"));
        // musicPlayer.startPlayingFile("/track001.mp3");
        break;
      case 2:

        break;
      case 3:

        break;

      default:
        Serial.println(F("Invalid option"));
    }



  }
  }  // end of loop()
  // end of receiver code

```

## Project Update (06/12)

Over the past week, we faced many difficulties with our robot. The robot would keep falling down, which is why we had to extend the wooden part on the back of the robot. We accidentally damaged the radio and the H-bridge motor, which is why we had to replace them. And finally, the radio and the transmitted on the robot weren't working, which is why the boards they were attached to had to be resoldered. However, with persistence, we managed to fix all the issues, and now our robot is functioning and we are ready to implement all the states for its participation in the play.

Video: https://drive.google.com/file/d/1iTT2iyWPqV-1_xf0btpR_F6FUnVKLW-N/view?usp=sharing

Going forward, we still need to extend the wooden part at the frontside of the robot to ensure stability during movement. We are also planning on updating all the states to make sure that they are synchronized with the script. 


