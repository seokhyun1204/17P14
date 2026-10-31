#include <Servo.h>

#define PIN_IR A0
#define PIN_LED 9
#define PIN_SERVO 10

#define _DUTY_MIN 544
#define _DUTY_NEU 1472
#define _DUTY_MAX 2400

#define _DIST_MIN 100.0
#define _DIST_MAX 250.0
#define EMA_ALPHA 0.2
#define LOOP_INTERVAL 25

Servo myservo;
unsigned long last_loop_time = 0;
float dist_prev = 175.0, dist_ema = 175.0;

void setup() {
  pinMode(PIN_LED, OUTPUT);
  myservo.attach(PIN_SERVO);
  myservo.writeMicroseconds(_DUTY_NEU);
  Serial.begin(1000000);
}

void loop() {
  if (millis() - last_loop_time < LOOP_INTERVAL) return;
  last_loop_time = millis();

  int a_value = analogRead(PIN_IR);
  float dist_raw = (a_value > 9)
    ? (6762.0 / (a_value - 9.0) - 4.0) * 10.0 - 60.0
    : -1.0;

  bool valid = dist_raw >= _DIST_MIN && dist_raw <= _DIST_MAX;
  digitalWrite(PIN_LED, valid ? HIGH : LOW);
  if (valid) dist_prev = dist_raw;

  dist_ema = EMA_ALPHA * dist_prev + (1 - EMA_ALPHA) * dist_ema;

  int duty = _DUTY_MIN + (dist_ema - _DIST_MIN)
    * (_DUTY_MAX - _DUTY_MIN) / (_DIST_MAX - _DIST_MIN);

  myservo.writeMicroseconds(duty);

  Serial.print("_DUTY_MIN:");  Serial.print(_DUTY_MIN);
  Serial.print(",_DIST_MIN:"); Serial.print(_DIST_MIN);
  Serial.print(",IR:");        Serial.print(a_value);
  Serial.print(",dist_raw:");  Serial.print(dist_raw);
  Serial.print(",ema:");       Serial.print(dist_ema);
  Serial.print(",servo:");     Serial.print(duty);
  Serial.print(",_DIST_MAX:"); Serial.print(_DIST_MAX);
  Serial.print(",_DUTY_MAX:"); Serial.println(_DUTY_MAX);
}
