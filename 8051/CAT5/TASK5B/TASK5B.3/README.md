# Continuous Lab Evaluation-5 

**EXP:** Continuous Lab Evaluation-5  
**DATE:** 10/08/2026  
**NAME OF EXP:** Hardware Timer and Interrupt-Based Timing for LED Fading, Pulse Generation, and Stepper Motor Control  
**HARDWARE SETUP / PLATFORM:** Keil uVision + 8051 Development Board (AT89S52, 11.0592 MHz) + 4x4 Matrix Keypad + Oscilloscope + Multimeter  
**TASK NO:** 2 - Waveform generation 

---

### Circuit Diagram
![Circuit Diagram](../CAT5B.png)

---

### Core Logic

* **1. Timer 1 Periodic Tick Base**
  Initializes Timer 1 in Mode 1 with `0xFC66` (`1 ms` tick) to drive a deterministic state counter (`wave_tick`) inside `timer1_ISR`.
  ```c
  TMOD = 0x11;
  TH1 = 0xFC;
  TL1 = 0x66;
  ET1 = 1;
  EA  = 1;
  TR1 = 1;
  ```

* **2. Multi-Waveform State Machine**
  Synthesizes four distinct digital waveforms inside `timer1_ISR` based on `waveform_mode`: a $50\%$ Square Wave, a $25\%$ Variable Duty-Cycle Wave, a `10000000` Pulse Train, and a 10-pulse Burst Signal followed by 10 silent ticks.
  ```c
  switch (waveform_mode) {
      case 1: OUTPUT = ~OUTPUT; break;                                   // Square Wave
      case 2: OUTPUT = (wave_tick < 2) ? 1 : 0; break;                   // Variable Duty-Cycle (25%)
      case 3: OUTPUT = (wave_tick == 0) ? 1 : 0; break;                  // Pulse Train (10000000)
      case 4: OUTPUT = (wave_tick < 10 && (wave_tick % 2 == 0)) ? 1 : 0; // Burst Signal
  }
  ```

* **3. Keypad Waveform Mode Switching**
  Reads keypad presses (`1`–`4`) in the main loop to update `waveform_mode` and reset `wave_tick` cleanly on each transition.
  ```c
  key = keypad_scan();
  if (key >= 1 && key <= 4) {
      waveform_mode = key;
      wave_tick = 0;
  }
  ```

---

### Inference
A single periodic Timer 1 interrupt successfully synthesized square, variable duty-cycle, pulse-train, and burst waveforms on the oscilloscope with clean transitions and zero blocking delays.