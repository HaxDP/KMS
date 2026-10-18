# ПР №6

## схема підключення

Arduino Uno, кнопка, світлодіод з резистором 220 Ом і потенціометр, який імітує датчик вологості

| елемент | виводи |
|---|---|
| кнопка | один вивід на піні 2, другий на GND, режим INPUT_PULLUP |
| світлодіод | через резистор 220 Ом на піні 8, катод на GND |
| потенціометр | VCC на 5V, SIG на A0, GND на GND |

## 

```cpp
const int buttonPin = 2, ledPin = 8, sensorPin = A0;
bool ledState = false, lastButtonState = HIGH;
unsigned long lastMeasurement = 0;
const unsigned long interval = 1000;

void setup()
{
  pinMode(buttonPin, INPUT_PULLUP);
  pinMode(ledPin, OUTPUT);
  Serial.begin(9600);
}

void loop()
{
  bool buttonState = digitalRead(buttonPin);
  if (buttonState == LOW && lastButtonState == HIGH)
  {
    ledState = !ledState;
    digitalWrite(ledPin, ledState);
  }
  lastButtonState = buttonState;

  unsigned long now = millis();
  if (now - lastMeasurement >= interval)
  {
    lastMeasurement = now;
    int value = map(analogRead(sensorPin), 0, 1023, 0, 100);
    Serial.print("sensor=hum;value=");
    Serial.println(value);
  }
}
```

значення АЦП 0-1023 переводиться функцією map у вологість 0-100 відсотків

## вивід монітора порту

формат рядка - sensor=hum;value=число

приклад рядків при положенні потенціометра приблизно посередині

```
sensor=hum;value=50
sensor=hum;value=50
sensor=hum;value=50
```

## висновки

millis кращий за delay бо не зупиняє програму, і кнопка та інші задачі опитуються на кожному проході loop, а вивід раз на секунду лише перевіряє чи минув інтервал

продуманий формат ключ=значення можна розібрати програмою, розбивши рядок по крапці з комою, а потім кожну пару по знаку рівності, тому дані легко передавати і читати далі мережею