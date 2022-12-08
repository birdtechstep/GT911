
# GT911
GT911 Touch library for Arduino

### This is experimental, only tested with an ESP32 & DFRobot DFR0669 TFT Touchscreen

Based off these repos
 - https://github.com/nik-sharky/arduino-goodix
 - https://github.com/u4mzu4/Arduino_GT911_Library

I needed a GT911 library to use with a [DFRobot TFT LCD Capacitive Touchscreen](https://www.dfrobot.com/product-2107.html) but as the RST pin is shared with SPI I couldn't use the existing libs.

It's been changed for my own needs so it can be used with polling instead of an interrupt.
The GT911 config can be read and written too but make sure you check the docs for values;

### Polling example
````cpp
#include <Arduino.h>
#include <GT911.h>

GT911 ts = GT911();
#define I2C_SDA_PIN 8
#define I2C_SCL_PIN 9
#define TP_INT_PIN  0
#define TP_RESET_PIN -1
#define I2C_CLOCK   400000L

void setup() {
  Serial.begin(115200);

  Wire.begin(I2C_SDA_PIN, I2C_SCL_PIN, I2C_CLOCK);
  Wire.beginTransmission(GT911_I2C_ADDR_28);
  byte error = Wire.endTransmission();
  ts.reset();
  if (error == 0) {
    ts.begin(TP_INT_PIN, TP_RESET_PIN, GT911_I2C_ADDR_28); //0x14
    Serial.println("Touch GT911 Address : 0x14");
  } else {
    ts.begin(TP_INT_PIN, TP_RESET_PIN, GT911_I2C_ADDR_BA); //0x5D
    Serial.println("Touch GT911 Address : 0x5D");
  }
}

void loop() {
  uint8_t touches = ts.touched(GT911_MODE_POLLING);

  if (touches) {
    GTPoint* tp = ts.getPoints();
    for (uint8_t  i = 0; i < touches; i++) {
      Serial.printf("#%d  %d,%d s:%d\n", tp[i].trackId, tp[i].x, tp[i].y, tp[i].area);
    }
  }
}
````

### Config update example
````cpp
void setup() {
  GTConfig *cfg = ts.readConfig();
  cfg->hSpace = (5 | (5 << 4));
  cfg->vSpace = (5 | (5 << 4));
  ts.writeConfig();
}
````
