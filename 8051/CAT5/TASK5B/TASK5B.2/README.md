# Continuous Lab Evaluation-5 

**EXP:** Continuous Lab Evaluation-5  
**DATE:** 10/08/2026  
**NAME OF EXP:** Hardware Timer and Interrupt-Based Timing for LED Fading, Pulse Generation, and Stepper Motor Control  
**HARDWARE SETUP / PLATFORM:** Keil uVision + 8051 Development Board (AT89S52, 11.0592 MHz) + 4x4 Matrix Keypad + Oscilloscope + Multimeter  
**TASK NO:** 2 - Variable Frequency 

---

### Circuit Diagram
![Circuit Diagram](../../CAT5B.png)

---

### Core Logic

* **1. Frequency Mode Reload Selection**
  Maps keypad selections (`1`–`5`) to 16-bit Timer 1 reload values (`reload_high`, `reload_low`) corresponding to $100\text{ Hz}$, $500\text{ Hz}$, $1\text{ kHz}$, $2\text{ kHz}$, and $5\text{ kHz}$ square-wave outputs.
  ```c
  void set_frequency(unsigned char mode)
  {
      switch(mode)
      {
          case 1: reload_high = 0xEE; reload_low = 0x00; break; // 100 Hz
          case 2: reload_high = 0xFC; reload_low = 0x66; break; // 500 Hz
          case 3: reload_high = 0xFE; reload_low = 0x33; break; // 1 kHz
          case 4: reload_high = 0xFF; reload_low = 0x1A; break; // 2 kHz
          case 5: reload_high = 0xFF; reload_low = 0xA4; break; // 5 kHz
      }
  }
  ```

* **2. Interrupt-Driven Square-Wave Toggling**
  Reloads Timer 1 inside `timer1_ISR` and complements the output pin (`OUTPUT = ~OUTPUT`) on every overflow to maintain a $50\%$ duty cycle.
  ```c
  void timer1_ISR(void) interrupt 3 {
      TH1 = reload_high;
      TL1 = reload_low;
      OUTPUT = ~OUTPUT;
  }
  ```

* **3. Non-Blocking 4x4 Matrix Keypad Scanning**
  Sequentially drives each keypad row LOW and reads column inputs in `main()` to switch frequencies dynamically without interrupting timer execution.
  ```c
  for (row = 0; row < 4; row++) {
      P2 |= 0xF0;
      P2 &= ~(1 << rowPin[row]);
      for (col = 0; col < 4; col++) {
          if ((P2 & (1 << colPin[col])) == 0) return keypad[row][col];
      }
  }
  ```

---

### Inference
Timer 1 interrupts reliably generated $50\%$ duty-cycle square waves across $100\text{ Hz}$, $500\text{ Hz}$, $1\text{ kHz}$, $2\text{ kHz}$, and $5\text{ kHz}$ modes, with instantaneous frequency switching via non-blocking keypad polling verified on the oscilloscope.