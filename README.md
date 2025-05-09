# Dual-Axis-Solar-Tracker
Developed an automated solar tracking system using self made quadrant detector with LDR sensors, servo motors to optimize panel alignment with sunlight on both horizontal and vertical axes. Improved energy efficiency by dynamically adjusting orientation for maximum solar intensity.
#include <Servo.h>

Servo verticalServo;
Servo horizontalServo;

// LDR Pins
const int ldrBottomLeft = A1;
const int ldrTopLeft = A0;
const int ldrTopRight = A3;
const int ldrBottomRight = A2;

// Servo Pins
const int vertServoPin = 4;
const int horizServoPin = 2;

// Settings
int threshold = 5;
int stepSize = 2;
int vertPos = 45;     // 0 to 90 range
int horizPos = 90;    // 0 to 180 range

void setup() {
  verticalServo.attach(vertServoPin);
  horizontalServo.attach(horizServoPin);

  verticalServo.write(vertPos);
  horizontalServo.write(horizPos);

  Serial.begin(9600);
}

void loop() {
  // Read LDRs
  int bl = analogRead(ldrBottomLeft);  // A1
  int tl = analogRead(ldrTopLeft);     // A0
  int tr = analogRead(ldrTopRight);    // A3
  int br = analogRead(ldrBottomRight); // A2

  Serial.print("A1: "); Serial.print(tl);
  Serial.print(" A0: "); Serial.print(bl);
  Serial.print(" A2: "); Serial.print(tr);
  Serial.print(" A3: "); Serial.println(br);

  // ----- Vertical Movement -----
  if (tl > bl + threshold) {
    vertPos -= stepSize;
    if (vertPos < 0) vertPos = 0;
    verticalServo.write(vertPos);
    Serial.print("Moving UP to: "); Serial.println(vertPos);
  } else if (bl > tl + threshold) {
    vertPos += stepSize;
    if (vertPos > 90) vertPos = 90;
    verticalServo.write(vertPos);
    Serial.print("Moving DOWN to: "); Serial.println(vertPos);
  } else {
    Serial.println("Vertical balanced — no move.");
  }

  // ----- Horizontal Movement with 180° range -----
  int leftDiff = abs(tl - bl);  // A1 - A0
  int rightDiff = abs(tr - br); // A3 - A2

  if (rightDiff > leftDiff + threshold) {
    horizPos += stepSize;
    if (horizPos > 180) horizPos = 180;
    horizontalServo.write(horizPos);
    Serial.print("Moving RIGHT to: "); Serial.println(horizPos);
  } else if (leftDiff > rightDiff + threshold) {
    horizPos -= stepSize;
    if (horizPos < 0) horizPos = 0;
    horizontalServo.write(horizPos);
    Serial.print("Moving LEFT to: "); Serial.println(horizPos);
  } else {
    Serial.println("Horizontal balanced — no move.");
  }

  delay(100); // Prevent jitter
}
