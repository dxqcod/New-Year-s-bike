#include <Adafruit_NeoPixel.h>
#include "LSM6DS3.h"
#include "Wire.h"

#define PIN1        3      // Первая лента
#define PIN2        6      // Вторая лента
#define NUM_LEDS    23     // Количество LED на каждой ленте
#define MAX_LIT     5      // Максимум горящих диодов одновременно

Adafruit_NeoPixel strip1(NUM_LEDS, PIN1, NEO_GRB + NEO_KHZ800);
Adafruit_NeoPixel strip2(NUM_LEDS, PIN2, NEO_GRB + NEO_KHZ800);

LSM6DS3 myIMU(I2C_MODE, 0x6A);

// === Настройки движения ===
const float MOVEMENT_THRESHOLD = 0.02;
const unsigned long CHECK_INTERVAL = 100;
const unsigned long INACTIVITY_TIMEOUT = 30000;  // 30 сек

// === Переменные для детекции движения ===
float prevX = 0.0, prevY = 0.0, prevZ = 0.0;
unsigned long lastCheckTime = 0;
unsigned long lastMovement = 0;
int movementCount = 0;

// === Эффекты ===
unsigned long effectSwitchTime = 0;
int currentEffect = 0;              // 0–7 (всего 8 эффектов)
int previousEffect = -1;            // Для отслеживания смены эффекта
const int TOTAL_EFFECTS = 8;
unsigned long previousMillis = 0;

void setup() {
  strip1.begin();
  strip2.begin();
  strip1.setBrightness(150);
  strip2.setBrightness(150);
  strip1.clear();
  strip2.clear();
  strip1.show();
  strip2.show();

  myIMU.begin();

  prevX = myIMU.readFloatAccelX();
  prevY = myIMU.readFloatAccelY();
  prevZ = myIMU.readFloatAccelZ() - 1.0;

  lastCheckTime = millis();
  lastMovement = millis();
  effectSwitchTime = millis();
}

void loop() {
  unsigned long currentMillis = millis();

  // Детекция движения
  if (currentMillis - lastCheckTime >= CHECK_INTERVAL) {
    lastCheckTime = currentMillis;

    float x = myIMU.readFloatAccelX();
    float y = myIMU.readFloatAccelY();
    float z = myIMU.readFloatAccelZ() - 1.0;

    float diffX = x - prevX;
    float diffY = y - prevY;
    float diffZ = z - prevZ;

    float change = sqrt(diffX * diffX + diffY * diffY + diffZ * diffZ);

    if (change > MOVEMENT_THRESHOLD) {
      movementCount++;
      if (movementCount >= 30) {
        lastMovement = currentMillis;
        movementCount = 0;
      }
    } else {
      movementCount = 0;
    }

    prevX = x;
    prevY = y;
    prevZ = z;
  }

  // Смена эффекта каждые 4 секунды
  if (currentMillis - effectSwitchTime >= 4000) {
    currentEffect = (currentEffect + 1) % TOTAL_EFFECTS;
    effectSwitchTime = currentMillis;

    // Если эффект сменился — очищаем ленты и сбрасываем состояние
    if (currentEffect != previousEffect) {
      clearBoth();
      showBoth();
      previousMillis = 0;        // Сброс таймера
      previousEffect = currentEffect;
    }
  }

  // Запуск эффектов
  if (currentMillis - lastMovement < INACTIVITY_TIMEOUT) {
    switch (currentEffect) {
      case 0: runningFire(strip1.Color(255, 0, 0), 40);   break; // Красный
      case 1: runningFire(strip1.Color(0, 255, 0), 40);   break; // Зелёный
      case 2: runningFire(strip1.Color(0, 0, 255), 40);   break; // Синий
      case 3: runningRainbowFire(40);                     break;
      case 4: randomSparkles(80);                         break;
      case 5: bounceBlock(strip1.Color(100, 255, 100), 40); break;
      case 6: twinkle(70);                                break;
      case 7: rainbowComet(50);                           break;
    }
  } else {
    clearBoth();
    showBoth();
  }

  delay(10);
}

// ====================== ЭФФЕКТЫ ======================

void runningFire(uint32_t color, int speedDelay) {
  if (millis() - previousMillis >= speedDelay) {
    previousMillis = millis();
    static int pos = 0;

    clearBoth();
    for (int i = 0; i < MAX_LIT; i++) {
      int p = (pos + i) % NUM_LEDS;
      strip1.setPixelColor(p, color);
      strip2.setPixelColor(p, color);
    }
    showBoth();
    pos = (pos + 1) % NUM_LEDS;
  }
}

void runningRainbowFire(int speedDelay) {
  if (millis() - previousMillis >= speedDelay) {
    previousMillis = millis();
    static int pos = 0;
    static uint16_t hue = 0;

    clearBoth();
    for (int i = 0; i < MAX_LIT; i++) {
      uint16_t pixelHue = hue + (i * 65536L / MAX_LIT);
      uint32_t color = strip1.gamma32(strip1.ColorHSV(pixelHue));
      int p = (pos + i) % NUM_LEDS;
      strip1.setPixelColor(p, color);
      strip2.setPixelColor(p, color);
    }
    showBoth();
    pos = (pos + 1) % NUM_LEDS;
    hue += 512;
  }
}

void randomSparkles(int speedDelay) {
  if (millis() - previousMillis >= speedDelay) {
    previousMillis = millis();

    clearBoth();
    int count = random(1, 5);
    for (int i = 0; i < count; i++) {
      int p = random(NUM_LEDS);
      strip1.setPixelColor(p, strip1.Color(255, 255, 255));
      strip2.setPixelColor(p, strip2.Color(255, 255, 255));
    }
    showBoth();
  }
}

void bounceBlock(uint32_t color, int speedDelay) {
  if (millis() - previousMillis >= speedDelay) {
    previousMillis = millis();
    static int pos = 0;
    static int dir = 1;

    clearBoth();
    for (int i = 0; i < MAX_LIT; i++) {
      int p = (pos + i) % NUM_LEDS;
      strip1.setPixelColor(p, color);
      strip2.setPixelColor(p, color);
    }
    showBoth();

    pos += dir;
    if (pos <= 0 || pos >= NUM_LEDS - MAX_LIT) dir = -dir;
  }
}

void twinkle(int speedDelay) {
  if (millis() - previousMillis >= speedDelay) {
    previousMillis = millis();

    clearBoth();
    int count = random(1, 5);
    for (int i = 0; i < count; i++) {
      int p = random(NUM_LEDS);
      strip1.setPixelColor(p, strip1.Color(255, 255, 200));
      strip2.setPixelColor(p, strip2.Color(255, 255, 200));
    }
    showBoth();
  }
}

void rainbowComet(int speedDelay) {
  if (millis() - previousMillis >= speedDelay) {
    previousMillis = millis();
    static int pos = 0;
    static uint16_t hue = 0;

    clearBoth();
    for (int i = 0; i < MAX_LIT; i++) {
      uint16_t h = hue + i * 3000;
      uint32_t c = strip1.gamma32(strip1.ColorHSV(h));
      int p = (pos - i + NUM_LEDS) % NUM_LEDS;
      strip1.setPixelColor(p, c);
      strip2.setPixelColor(p, c);
    }
    showBoth();
    pos = (pos + 1) % NUM_LEDS;
    hue += 800;
  }
}

// Вспомогательные функции
void clearBoth() {
  strip1.clear();
  strip2.clear();
}

void showBoth() {
  strip1.show();
  strip2.show();
}
