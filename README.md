# Starry Night Project Using Arduino Nano and Optical Fibres

A DIY starry-night effect built with an **Arduino Nano**, **WS2812B RGB LEDs**, and **optical fibres**. A 64-bit Linear Feedback Shift Register (LFSR) generates changing brightness and timing to create a natural twinkling effect, while optical fibres carry light from the LEDs to star-like points on a cardboard surface.

## Project Overview

The project creates a glowing star field using inexpensive materials and a simple Arduino-controlled LED setup. Each WS2812B LED feeds multiple optical fibres, allowing one LED to illuminate several star openings. Different fibres may capture different RGB colour components, producing a mix of white, red, green, and blue points even when the LEDs are programmed to emit white light.

## Features

- Randomized twinkling effect using a 64-bit LFSR
- WS2812B RGB LED control with the FastLED library
- Optical fibres distribute light across the front panel
- Compact cardboard structure with rear-mounted electronics
- Detachable LED connection using a 3-pin JST connector

## Components Required

- Arduino Nano
- WS2812B LED strip (14 LEDs)
- 12 × 12 cm cardboard sheet with an additional 2-inch margin on one side
- Small cardboard pieces for rear support
- Zero board
- 3-pin JST male-female connector
- 470 Ω resistor
- Optical fibres
- Two-part epoxy adhesive
- USB cable
- Paper knife and compass

## Circuit Connections

| WS2812B LED Strip | Arduino Nano |
|---|---|
| VCC | 5V |
| GND | GND |
| DIN | D4 through a 470 Ω resistor |

**Note:** The report specifies powering the LED strip from the Arduino Nano's 5V pin and limits the FastLED power setting to 500 mA. Confirm that the power source and wiring can safely support the LED load before operating the project.

## Construction

1. Prepare a 12 × 12 cm cardboard sheet, leaving an additional 2-inch margin on one side for mounting the zero board.
2. Mark a 10 × 10 cm central grid for the star openings.
3. Make the holes using a compass and trim the sheet edges neatly with a paper knife.
4. Glue small rectangular cardboard supports to the back along all four sides.
5. Attach the WS2812B LED strip to the cardboard supports using epoxy.
6. Glue approximately 20–24 optical fibres to each LED so that one LED can illuminate multiple star openings.
7. Attach the support assembly, LEDs, and fibres to the back of the main cardboard sheet.
8. Glue the free ends of the optical fibres into the front-side holes.
9. Use shorter fibres for nearby holes and longer fibres for farther holes.
10. Place 2–4 fibres in each hole, depending on its size and the desired brightness.
11. Do the gluing from the back so the front surface stays smooth and clean.
12. Assemble a zero board with the Arduino Nano, 3-pin JST connector, and 470 Ω resistor, then mount it on the extra margin.
13. Use the JST connector to make the LED strip detachable.
14. Make an opening near the zero board for the USB cable used to power and program the Arduino.

## Software

### Requirements

- Arduino IDE
- [FastLED library](https://github.com/FastLED/FastLED)
- Arduino Nano board support

### How It Works

1. The Arduino samples the least significant bit of analog readings from pin `A0` to initialize the LFSR.
2. A fallback seed is used if the LFSR state is zero.
3. FastLED is configured for 14 WS2812B LEDs on digital pin `D4`.
4. The program sets the LEDs to white, then scales each LED's brightness using the low byte of the LFSR state.
5. The LFSR is advanced using XOR feedback taps, and a changing delay varies the twinkling timing.

### Arduino Code

Save the following as an `.ino` sketch and upload it to your Arduino Nano:

```cpp
#include <FastLED.h>
#include <stdint.h>

#define NUM_LEDS 14
#define LED_PIN 4

uint64_t lfsr;
uint8_t bit;
CRGB leds[NUM_LEDS];

void setup() {
  lfsr = 0;

  for (int i = 0; i < 64; i++) {
    int noise = analogRead(A0);
    lfsr = (lfsr << 1) | (noise & 1);
    delay(20);
  }

  if (lfsr == 0) {
    lfsr = (0xACE1ULL << 48) | 0x1234ULL;
  }

  FastLED.addLeds<WS2812B, LED_PIN, GRB>(leds, NUM_LEDS);
  FastLED.setMaxPowerInVoltsAndMilliamps(5, 500);
  FastLED.setBrightness(255);
}

void loop() {
  for (int i = 0; i < NUM_LEDS; i++) {
    leds[i] = CRGB(255, 255, 255);

    bit = ((lfsr >> 0) ^ (lfsr >> 1) ^
           (lfsr >> 3) ^ (lfsr >> 4)) & 1;

    lfsr = (lfsr >> 1) | ((uint64_t)bit << 63);

    leds[i].nscale8(lfsr & 0xFF);

    FastLED.show();
    delay(lfsr & 0x3F);
  }
}
```

## Results and Observations

- The optical fibres transmitted light from the LEDs to the front surface, creating a starry-night effect.
- Each LED was connected to approximately 20–24 fibres.
- Some fibres captured different RGB colour components, creating white, red, green, and blue stars.
- Rear-mounted supports helped maintain LED alignment and distribute light.
- The JST connector and rear-side wiring made the setup cleaner and easier to service.

## Possible Improvements

- Add animated RGB colour effects.
- Add sound-reactive lighting.
- Experiment with fibre placement and the number of fibres per star to vary brightness and colour.

## License

No license has been specified yet. Add a `LICENSE` file if you want others to reuse, modify, or distribute this project under defined terms.
