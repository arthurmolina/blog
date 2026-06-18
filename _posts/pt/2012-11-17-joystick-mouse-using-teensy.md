---
title: "Mouse com joystick usando Teensy"
lang: pt
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

Tenho feito algumas brincadeiras com computação física e aceitei o meu primeiro projeto de um dispositivo usando microprocessador. Enquanto o componente que preciso pra terminar esse projeto não chega (fiz o pedido pelo MercadoLivre) decidi brincar com outros componentes. Como eu estou fazendo um dispositivo que vai simular alguns comandos de teclado, decidi usar o Teensy 2.0 que possibilita uma programação de HID de modo bem fácil. É incrível como é fácil fazer isso com o Teensy.

Aproveitei um módulo de joystick que já tinha e adaptei o controle do mouse.

Então aqui vai a receita:

# Ingredientes

- Teensy 2.0
- Módulo Joystick KY-023
- alguns fios
- Adaptador USB

# O Código

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
  Serial.println("—-");
  Serial.print(v1);
  Serial.print(" – ");
  Serial.println(change(v1, v1Padrao, 1023));
  Serial.print(v2);
  Serial.print(" – ");
  Serial.println(change(v2, v2Padrao, 997) );
  Mouse.move( change(v1, 506, 1023) , change(v2, 512, 997) );
  delay(100);
}
```

# O Esquema

{% include figure image_path="/assets/images/2012/joystick-mouse-using-teensy.png" alt="o Esquema" %}

# Algumas explicações

O joystick nada mais é do que dois potenciômetros e um botão (para quando você pressiona o joystick para baixo). Os terminais VRx e VRy são conectados às portas analógicas do Teensy, enquanto o terminal SW, que é o botão, é conectado a uma porta digital (embora não esteja sendo usado). As portas analógicas variam de zero a 1023. Porém, após alguns testes descobri que em um eixo os valores vão de zero a 1023, parando em 506 quando centralizado, e o outro eixo vai de zero a 997, parando em 512. Certamente outras peças terão limites diferentes. A função change adapta o valor retornado pelos potenciômetros para o número mais interessante para a função do mouse, que deve variar de -127 a 127.

# Problemas

Tentei fazer o botão que vem junto funcionar como clique do mouse, porém não sei por que às vezes o valor é zero e outras vezes é 1 sem nem mover o joystick. Outro problema é que nem sempre move na direção certa. Não entendi o motivo. Se alguém souber, por favor me explique.

# O resultado

{% include figure image_path="/assets/images/2012/joystick-mouse-using-teensy2.png" alt="o resultado" %}
