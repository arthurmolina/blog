---
title: "Joystick mouse using Teensy"
lang: en
last_modified_at: 2012-11-17T16:00:00-03:00
categories:
  - articles
tags:
  - code
  - electronics
  - arduino
show_overlay_excerpt: false
toc: true
header:
  image: /assets/images/nonsense.jpg
  overlay_image: /assets/images/nonsense.jpg
  overlay_filter: 0.15
  show_overlay_excerpt: false
---

I've been playing around with physical computing and accepted my first project for a device using a microprocessor. While the component I need to finish this project isn't enough (I ordered it through MercadoLivre) I decided to play with other components. As I'm making a device that will simulate some keyboard commands, I decided to use the Teensy 2.0 which allows for very easy HID programming. It's amazing how easy it is to do this with Teensy.

I took advantage of a joystick module I already had and adapted the mouse control.

So here's the recipe:

# Ingredients

- Teensy 2.0
- Joystick KY-023 Module
- some wire
- USB adapter

# The Code

```cpp
int joyPin1 = PIN_F0;
int joyPin2 = PIN_F1;
int v1 = 0;
int v2 = 0;
int v1Padrao = 0;
int v2Padrao = 0;
int passo = 20;

void setup() {
  Serial.begin(9600);
  v1Padrao = analogRead(joyPin1);
  delay(100);
  v2Padrao = analogRead(joyPin2);
}

int change(int v, int unchange, int max_state) {
  if(v > unchange) {
    return int(passo * ( v – unchange ) / (max_state – unchange));
  } else if (v < unchange) {
    return int( (v / unchange * passo) – passo );
  }
  return 0;
}

void loop() {
  v1 = analogRead(joyPin1);
  delay(100);
  v2 = analogRead(joyPin2);
  Serial.println(“—-“);
  Serial.print(v1);
  Serial.print(” – “);
  Serial.println(change(v1, v1Padrao, 1023));
  Serial.print(v2);
  Serial.print(” – “);
  Serial.println(change(v2, v2Padrao, 997) );
  Mouse.move( change(v1, 506, 1023) , change(v2, 512, 997) );
  delay(100);
}
```

# The Schematics

{% include figure image_path="/assets/images/2012/joystick-mouse-using-teensy.png" alt="the Schematics" %}

# Some explanations

The Joystick is nothing more than two potentiometers and a button (in case you press the joystick down). The VRx and VRy terminals are connected to the Teensy's analog ports while the SW terminal, which is the button, is connected to a digital port (although it is not being used). The analog ports range from Zero to 1023. However, after some tests I discovered that on one axis the values range from zero to 1023, stopping at 506 when it is centered, and the other axis goes from zero to 997, stopping at 512. Certainly other pieces will have different limits. The change function adapts the value returned by the potentiometers to the most interesting number for the mouse function, which should range from -127 to 127.

# Problems

I tried to make the button that comes with it work as a mouse click, however I don't know why sometimes the value is zero and other times it is 1 without even moving the joystick. Another issue is that it doesn't always move in the right direction. I didn't understand the reason. If anyone knows, please explain to me.

# Here is the result

{% include figure image_path="/assets/images/2012/joystick-mouse-using-teensy2.png" alt="the results" %}