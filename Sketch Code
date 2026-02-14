#include <Servo.h>

Servo esc;
const int ESC_PIN = 9;

// Pulse widths (microseconds)
// Most ESCs use: 1000 = min, 1500 = mid, 2000 = max
// Calibrate these for your ESC if needed.
const int PULSE_MIN = 1000;
const int PULSE_MID = 1500;
const int PULSE_MAX = 2000;

// We will NEVER hit max speed:
// forward tops out below max
const int FWD_TOP = 1500 + 220;   // 1720us (example safe top)
const int REV_TOP = 1500 - 220;   // 1280us (example safe top for reverse)

// Smooth ramp settings
const int STEP_US = 4;            // how much pulse changes each step
const int STEP_DELAY_MS = 12;     // delay between steps (controls ramp smoothness)

// If your ESC is NOT bidirectional, set this false.
// If your ESC supports reverse (bidirectional), set true.
const bool BIDIRECTIONAL_ESC = true;

void writePulse(int us) {
  us = constrain(us, PULSE_MIN, PULSE_MAX);
  esc.writeMicroseconds(us);
}

void ramp(int startUs, int endUs) {
  if (startUs < endUs) {
    for (int p = startUs; p <= endUs; p += STEP_US) {
      writePulse(p);
      delay(STEP_DELAY_MS);
    }
  } else {
    for (int p = startUs; p >= endUs; p -= STEP_US) {
      writePulse(p);
      delay(STEP_DELAY_MS);
    }
  }
}

// Runs a phase for durationMs:
// - ramps up to targetUs
// - holds briefly
// - ramps back down to baselineUs
void runPhase(int baselineUs, int targetUs, unsigned long durationMs) {
  // How long ramp takes:
  unsigned long rampTime = (unsigned long)(abs(targetUs - baselineUs) / STEP_US) * STEP_DELAY_MS;

  // If duration is small, keep ramps smaller
  unsigned long holdTime = 0;
  if (durationMs > (2 * rampTime)) {
    holdTime = durationMs - (2 * rampTime);
  } else {
    // Not enough time for full ramp; shorten by reducing target a bit:
    // (simple approach: just do half-ramp up then half-ramp down)
    rampTime = durationMs / 2;
    holdTime = 0;
  }

  // Ramp up
  ramp(baselineUs, targetUs);

  // Hold
  if (holdTime > 0) delay(holdTime);

  // Ramp down
  ramp(targetUs, baselineUs);
}

void armESC() {
  // Common arming: send minimum for a few seconds
  writePulse(PULSE_MIN);
  delay(3000);

  // For bidirectional ESCs, sometimes neutral is required:
  if (BIDIRECTIONAL_ESC) {
    writePulse(PULSE_MID);
    delay(2000);
  }
}

void setup() {
  esc.attach(ESC_PIN);

  // Safety startup
  writePulse(PULSE_MIN);
  delay(1000);

  armESC();
}

void loop() {
  // 1) REST 2 seconds
  // If bidirectional ESC: use mid (neutral). If not: use min (stop).
  if (BIDIRECTIONAL_ESC) writePulse(PULSE_MID);
  else writePulse(PULSE_MIN);
  delay(2000);

  // 2) "CLOCKWISE" 3 seconds (ramp up + ramp down), never max
  // - bidirectional: baseline = mid, forward = above mid
  // - one-way: baseline = min, forward = above min
  if (BIDIRECTIONAL_ESC) {
    runPhase(PULSE_MID, FWD_TOP, 3000);
  } else {
    // For one-way ESC, choose a safe range between MIN and not-max:
    int baseline = PULSE_MIN;
    int oneWayTop = PULSE_MIN + 350; // e.g., 1350us, adjust for your ESC
    runPhase(baseline, oneWayTop, 3000);
  }

  // 3) "ANTICLOCKWISE" 4 seconds (ramp up + ramp down), never max
  // Only works if ESC truly supports reverse.
  if (BIDIRECTIONAL_ESC) {
    // Some bidirectional ESCs need a brief neutral before reversing
    writePulse(PULSE_MID);
    delay(250);

    runPhase(PULSE_MID, REV_TOP, 4000);
  } else {
    // Not possible to reverse on a standard ESC
    // We'll just do another gentle ramp as a placeholder:
    int baseline = PULSE_MIN;
    int oneWayTop = PULSE_MIN + 300;
    runPhase(baseline, oneWayTop, 4000);
  }

  // Small neutral/stop gap to keep things stable between cycles
  if (BIDIRECTIONAL_ESC) writePulse(PULSE_MID);
  else writePulse(PULSE_MIN);
  delay(300);
}
