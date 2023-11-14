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

Additionally, we finalized the prototype of the mouth movement for the pipe mechanism. Now going forward, we will change the popsicle sticks we are using to aluminium. Additionally, we will create another layer with a hole that guides the pipe and prevents it from moving up and down. Finally, we will create holes in the wooden parts of the robot to make sure that the wires are connected to the base. 

