**Component tutorial** for 1.14" IPS LCD (ST7789V driver), including full pin descriptions, initialization, and practical examples:

---

# **Complete ST7789V LCD Module Guide**

## **1. Module Specifications**
| Parameter          | Value               |
|--------------------|---------------------|
| Display Type       | IPS LCD             |
| Resolution         | 135x240 pixels      |
| Driver IC          | ST7789V             |
| Interface          | 4-wire SPI          |
| Viewing Angles     | 170° horizontal/vertical |
| Power Supply       | 3.3V-5V            |
| Backlight          | LED (PWM controllable) |

## **2. Pinout Description**
### **Full Pin Functions**
| Pin Label | Pin Type | Description                  | Arduino Connection |
|-----------|----------|------------------------------|--------------------|
| **VCC**   | Power    | 3.3V-5V power input          | 3.3V or 5V         |
| **GND**   | Ground   | Common ground                | GND                |
| **SCL**   | Input    | SPI clock signal             | D13 (SCK)          |
| **SDA**   | I/O      | SPI data input/output        | D11 (MOSI)         |
| **RES**   | Input    | Reset (active low)           | D8 (or tie to VCC) |
| **DC**    | Input    | Data/Command selector        | D9                 |
| **CS**    | Input    | Chip Select (active low)     | D10                |
| **BLK**   | Input    | Backlight control (PWM)      | D3 (PWM capable)   |

### **Critical Pin Details**
- **RES (Reset)**: Minimum 10μs low pulse required for hardware reset
- **DC (Data/Command)**:
  - HIGH = Data mode (pixel/color data)
  - LOW = Command mode (register configuration)
- **BLK (Backlight)**:
  - 3.3V = Maximum brightness
  - PWM = Adjustable brightness (0-255)
  - GND = Backlight off (screen still active)

## **3. Electrical Characteristics**
| Parameter          | Min | Typical | Max | Unit |
|--------------------|-----|---------|-----|------|
| Operating Voltage | 3.0 | 3.3     | 5.5 | V    |
| Current (Display) | 15  | 25      | 40  | mA   |
| Current (Backlight)| 20  | 30      | 50  | mA   |
| SPI Clock Speed   | -   | 40      | 80  | MHz  |

## **4. Initialization Sequence**
### **Hardware Initialization**
1. Power up sequence:
   - Apply VCC (3.3V)
   - Hold RES low for 10μs, then high
   - Wait 120ms before sending commands

2. Software initialization:
```cpp
Adafruit_ST7789 tft(TFT_CS, TFT_DC, TFT_RST);
void setup() {
  tft.init(135, 240, SPI_MODE3); // Width, height, SPI mode
  tft.setRotation(1); // Adjust orientation
  tft.fillScreen(ST77XX_BLACK);
}
```

### **SPI Configuration**
```cpp
SPI.beginTransaction(SPISettings(
  40000000, // 40MHz clock
  MSBFIRST, // Bit order
  SPI_MODE3 // Clock polarity/phase
));
```

## **5. Complete Function Reference**
### **Display Control**
| Function | Parameters | Description |
|----------|------------|-------------|
| `init()` | width, height [, mode] | Initialize display |
| `setRotation()` | 0-3 | Set display orientation |
| `invertDisplay()` | true/false | Invert colors |
| `enableDisplay()` | true/false | Turn display on/off |

### **Drawing Primitives**
```cpp
// Pixel operations
tft.drawPixel(x, y, color);
uint16_t color = tft.readPixel(x, y);

// Shape drawing (x,y positions in pixels)
tft.drawLine(x0,y0, x1,y1, color);
tft.drawRect(x,y, w,h, color);
tft.fillRect(x,y, w,h, color);
tft.drawCircle(x,y,r, color);
tft.fillCircle(x,y,r, color);
tft.drawTriangle(x1,y1, x2,y2, x3,y3, color);
```

### **Text Rendering**
```cpp
// Font control
tft.setTextSize(n);      // 1-8x scaling
tft.setTextColor(color); // Foreground
tft.setTextColor(fg, bg);// With background
tft.setTextWrap(true);   // Auto newline

// Position cursor and print
tft.setCursor(x, y);
tft.print("Hello");
tft.println(123.45, 2);  // Print float with 2 decimals
```

## **6. Practical Example**
### **Sensor Dashboard**
```cpp
#include <Adafruit_ST7789.h>
#define TFT_CS   10
#define TFT_DC   9
#define TFT_RST  8

Adafruit_ST7789 tft(TFT_CS, TFT_DC, TFT_RST);

void setup() {
  tft.init(135, 240);
  tft.setRotation(1);
  tft.fillScreen(ST77XX_BLACK);
  analogWrite(3, 128); // 50% backlight
}

void loop() {
  // Read sensors
  float temp = readTemperature();
  int humidity = readHumidity();
  
  // Update display
  tft.fillRect(0, 0, 135, 60, ST77XX_BLUE);
  tft.setTextColor(ST77XX_WHITE);
  tft.setCursor(10, 10);
  tft.print("Temp: "); tft.print(temp); tft.print("C");
  
  tft.setCursor(10, 30);
  tft.print("Humidity: "); tft.print(humidity); tft.print("%");
  
  delay(1000);
}
```

## **7. Troubleshooting Guide**
| Symptom | Solution |
|---------|----------|
| Blank screen | Check RES pulse, verify SPI pins |
| Flickering | Reduce SPI speed to 30MHz |
| Wrong colors | Confirm SPI_MODE3, check DC pin |
| Backlight not working | Test BLK with direct 3.3V connection |
| Garbled text | Use `tft.setRotation()` to adjust orientation |

## **8. Advanced Techniques**
### **Partial Updates**
```cpp
tft.setAddrWindow(x,y,w,h);
for(int i=0; i<w*h; i++) {
  tft.writePixel(calculateColor());
}
```

### **Double Buffering**
```cpp
uint16_t buffer[135*240]; // Full screen buffer
// Draw to buffer first, then:
tft.startWrite();
tft.writePixels(buffer, 135*240);
tft.endWrite();
```

This complete guide covers all aspects of working with your ST7789V display. For best results:
1. Start with basic wiring verification
2. Use the test pattern code
3. Gradually implement your application features
4. Optimize performance as needed
