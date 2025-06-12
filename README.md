#define TRIG_PIN 9
#define ECHO_PIN 10

#define IN1 4
#define IN2 5
#define IN3 6
#define IN4 7

#define LEFT_IR_PIN 2  // Left IR sensor pin
#define RIGHT_IR_PIN 3 // Right IR sensor pin

// Threshold distances in cm
#define MIN_DISTANCE 10       // Go backward if the obstacle is closer than this
#define FOLLOW_DISTANCE 20    // Follow obstacle if within this range

void setup() {
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);

  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  pinMode(LEFT_IR_PIN, INPUT);
  pinMode(RIGHT_IR_PIN, INPUT);

  Serial.begin(9600); // For debugging
}

void loop() {
  long duration = getUltrasonicDistance();
  int distance = duration * 0.034 / 2; // Convert to cm

  bool leftDetected = digitalRead(LEFT_IR_PIN) == LOW;  // Object detected on the left
  bool rightDetected = digitalRead(RIGHT_IR_PIN) == LOW; // Object detected on the right

  Serial.print("Distance: ");
  Serial.println(distance);
  Serial.print("Left IR: ");
  Serial.println(leftDetected);
  Serial.print("Right IR: ");
  Serial.println(rightDetected);

  if (distance > 0 && distance < MIN_DISTANCE) {
    goBackward(); // Move backward briefly for safety
  } else if (leftDetected && !rightDetected) {
    turnLeft(); // Turn left to follow the object
  } else if (!leftDetected && rightDetected) {
    turnRight(); // Turn right to follow the object
  } else if (distance >= MIN_DISTANCE && distance < FOLLOW_DISTANCE) {
    followObstacle(); // Follow obstacle
  } else {
    stopMotors(); // Stop if no obstacle is detected
  }
  delay(100);
}

long getUltrasonicDistance() {
  // Send a 10us pulse to TRIG pin
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  // Measure the time for the echo
  return pulseIn(ECHO_PIN, HIGH);
}

void followObstacle() {
  // Move forward
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void goBackward() {
  // Move backward
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  delay(50); // Move backward for 50ms
  stopMotors();
}

void turnLeft() {
  // Turn left by stopping right motor and moving left motor
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
  delay(100); // Adjust delay as needed for a smooth turn
}

void turnRight() {
  // Turn right by stopping left motor and moving right motor
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
  delay(100); // Adjust delay as needed for a smooth turn
}

void stopMotors() {
  // Stop the motors
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
}
