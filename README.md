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
