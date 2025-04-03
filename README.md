# **Complete ST7789V LCD Tutorial for Arduino**

## **1. Library Functions Deep Dive**

### **Initialization Parameters**
```cpp
tft.init(width, height, spiMode, frequency);
```
- **width**: Display width in pixels (135)
- **height**: Display height in pixels (240)
- **spiMode** (optional):
  - `SPI_MODE0` to `SPI_MODE3` (try `SPI_MODE3` if display glitches)
- **frequency** (optional): SPI clock speed (default 40MHz)

### **Rotation Settings**
```cpp
tft.setRotation(rotation);
```
| Value | Orientation |
|-------|-------------|
| 0 | Portrait (0°) |
| 1 | Landscape (90°) |
| 2 | Inverse Portrait (180°) |
| 3 | Inverse Landscape (270°) |

## **2. Core Drawing Functions**

### **Pixel Manipulation**
```cpp
// Draw single pixel
tft.drawPixel(x, y, color);

// Read pixel color (if supported)
uint16_t color = tft.readPixel(x, y);
```

### **Shape Drawing**
```cpp
// Rectangles
tft.drawRect(x, y, width, height, color);  // Outline
tft.fillRect(x, y, width, height, color);  // Filled

// Circles
tft.drawCircle(x, y, radius, color);      // Outline
tft.fillCircle(x, y, radius, color);      // Filled

// Triangles
tft.drawTriangle(x1,y1, x2,y2, x3,y3, color);
```

### **Text Display**
```cpp
tft.setTextColor(color);          // Foreground
tft.setTextColor(fg, bg);         // Foreground+Background
tft.setTextSize(scale);           // 1-8 multiplier
tft.setTextWrap(true/false);      // Auto line-wrap
tft.setCursor(x, y);              // Position
tft.print("Hello");               // Print text
tft.println("World!");            // Print with newline
```

## **3. Color System**

### **Predefined Colors**
```cpp
ST77XX_BLACK   ST77XX_WHITE
ST77XX_RED     ST77XX_GREEN
ST77XX_BLUE    ST77XX_CYAN
ST77XX_MAGENTA ST77XX_YELLOW
```

### **Custom 16-bit Colors**
```cpp
// RGB565 format (5-6-5 bits)
uint16_t myColor = tft.color565(r, g, b);
// Where: r(0-31), g(0-63), b(0-31)
```

## **4. Advanced Features**

### **Partial Screen Updates**
```cpp
tft.setAddrWindow(x, y, w, h);  // Define area
tft.pushColor(color, count);    // Fill area
```

### **Display Control**
```cpp
tft.invertDisplay(true);   // Invert colors
tft.enableDisplay(true);   // Turn on/off
tft.setBrightness(128);    // PWM control (0-255)
```

## **5. Performance Optimization**

### **Double Buffering**
```cpp
// Create off-screen buffer
uint16_t buffer[135 * 240];
tft.startWrite();
tft.writePixels(buffer, 135*240);
tft.endWrite();
```

### **Fast Drawing Tricks**
```cpp
tft.startWrite();  // Begin transaction
// Multiple draw operations...
tft.endWrite();    // End transaction
```

## **6. Complete Example Code**

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_ST7789.h>
#include <SPI.h>

#define TFT_CS   10
#define TFT_DC   9
#define TFT_RST  8

Adafruit_ST7789 tft(TFT_CS, TFT_DC, TFT_RST);

void setup() {
  Serial.begin(115200);
  
  // Initialize display
  if (!tft.init(135, 240)) {
    Serial.println("Display init failed!");
    while(1);
  }
  
  tft.setRotation(1);
  tft.fillScreen(ST77XX_BLACK);
  
  // Display info
  tft.setTextColor(ST77XX_GREEN);
  tft.setTextSize(1);
  tft.setCursor(0, 0);
  tft.println("ST7789V LCD Demo");
  
  // Draw shapes
  tft.drawRect(10, 30, 50, 30, ST77XX_RED);
  tft.fillCircle(90, 45, 15, ST77XX_BLUE);
  
  // Custom color
  uint16_t purple = tft.color565(31, 0, 31);
  tft.fillTriangle(30, 90, 60, 90, 45, 120, purple);
}

void loop() {
  // Update sensor data
  tft.fillRect(0, 130, 135, 30, ST77XX_BLACK);
  tft.setCursor(10, 140);
  tft.print("Voltage: ");
  tft.print(3.3 * analogRead(A0)/1024, 2);
  tft.print("V");
  
  delay(1000);
}
```

## **7. Input Handling**

### **Button Integration**
```cpp
#define BUTTON_PIN 2

void setup() {
  pinMode(BUTTON_PIN, INPUT_PULLUP);
}

void loop() {
  if (digitalRead(BUTTON_PIN) == LOW) {
    tft.fillScreen(ST77XX_RED);
    tft.setCursor(10, 10);
    tft.println("Button Pressed!");
    delay(200);
  }
}
```

### **Touch Screen (if available)**
```cpp
// Requires additional touch library
#include <XPT2046_Touchscreen.h>
#define TOUCH_CS 4

XPT2046_Touchscreen ts(TOUCH_CS);

void setup() {
  ts.begin();
}

void loop() {
  if (ts.touched()) {
    TS_Point p = ts.getPoint();
    tft.fillCircle(p.x, p.y, 5, ST77XX_WHITE);
  }
}
```

## **8. Power Management**

### **Sleep Mode**
```cpp
tft.enableDisplay(false);  // Power saving
tft.enableDisplay(true);   // Wake up
```

### **Backlight Control**
```cpp
#define BL_PIN 3

void setup() {
  pinMode(BL_PIN, OUTPUT);
  analogWrite(BL_PIN, 128); // 50% brightness
}
```

This comprehensive tutorial covers all major aspects of working with the ST7789V display. For best results:
1. Start with basic test code
2. Gradually add features
3. Optimize for your specific application
4. Monitor memory usage on small MCUs
