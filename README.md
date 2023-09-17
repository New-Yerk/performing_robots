Journal for the class

09/13

Sketch of the robot
![WhatsApp Image 2023-09-13 at 14 20 13](https://github.com/New-Yerk/performing_robots/assets/78387567/6346b6fe-a17f-43eb-acc2-70940c983a46)


09/17

The robot base:

![IMG_0788](https://github.com/New-Yerk/performing_robots/assets/78387567/ad230808-6b98-4929-b845-4aedf3fe44bf)

Issues: when constructing the base, the only issue we faced was with the placement of the batteris and the arduino. We solved the issue by drilling a hole in the base and soldering the wires of the motors in a way that allowed us to put the wires through the hole and keep the battery and the motor on top of the base in an accessible way. We believe that the issue of the weight imbalance will be solved by the addition of a caster. 

Code: 

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

Reading Response:

“Chapter 7: Machines/Mechanicals”

What struck me about this chapter was the discussion regarding the conflict between the appearance and the performance of the robots. How does an object that is essentially “dead” can be relatable in front of a human audience? The author cites the opinion of Jasia Reichardt, a curator and art historian, who claimed that it is not the appearance, but the behavior of a robot is what allows us to “ascribe sentience and performativity” to the object. This point made me think of all the fictionalized material that involves a non-anthropomorphic robot such as Wall-E and the emotions they were able to convey to the audience, despite their appearance. Therefore, I came to a conclusion that as humans we relate to actions and the perceived emotions behind them, which is why puppets or toys can have such strong emotional presences, elevating the storytelling of the medium they are performing in. I interpret this as an invitation to experiment as much as I would like to with the shape or the appearance of the robot, because ultimately the audience would relate to the actions or the intentions behind the existence of that robot. 
