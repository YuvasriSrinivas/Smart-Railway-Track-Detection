# Smart-Railway-Track-Detection
A Realtime project with the concept of Railway Track Detection that based on the Internet Of Things.
#include <SoftwareSerial.h>

#define trig1 2
#define echo1 3
#define trig2 4
#define echo2 5
#define trig3 6
#define echo3 7
#define Motor_pin1 8
#define Motor_pin2 9
#define Motor_pin3 10
#define Motor_pin4 11
#define Buzzer A0
#define GSM_TX 12
#define GSM_RX 13

SoftwareSerial gsmSerial(GSM_RX, GSM_TX);

const String PHONE_NUMBER = "+918317597089";
unsigned long lastSMSTime = 0;
const unsigned long SMS_COOLDOWN = 30000;

bool rightCrackAlerted = false;
bool leftCrackAlerted = false;
bool frontObstacleAlerted = false;

void setup() {
  Serial.begin(9600);
  gsmSerial.begin(9600);

  pinMode(Buzzer, OUTPUT);
  digitalWrite(Buzzer, LOW);

  pinMode(trig1, OUTPUT);
  pinMode(echo1, INPUT);
  pinMode(trig2, OUTPUT);
  pinMode(echo2, INPUT);
  pinMode(trig3, OUTPUT);
  pinMode(echo3, INPUT);

  pinMode(Motor_pin1, OUTPUT);
  pinMode(Motor_pin2, OUTPUT);
  pinMode(Motor_pin3, OUTPUT);
  pinMode(Motor_pin4, OUTPUT);

  digitalWrite(Motor_pin1, HIGH);
  digitalWrite(Motor_pin2, LOW);
  digitalWrite(Motor_pin3, HIGH);
  digitalWrite(Motor_pin4, LOW);

  Serial.println("Obstacle Detection Robot with GSM Alert System Initialized");

  delay(2000);
  initGSM();
}

void loop() {
  int distance1 = getDistance(trig1, echo1);
  Serial.print("Distance1 (Right): ");
  Serial.print(distance1);
  Serial.println(" cm");
  delay(100);

  int distance2 = getDistance(trig2, echo2);
  Serial.print("Distance2 (Left): ");
  Serial.print(distance2);
  Serial.println(" cm");
  delay(100);

  int distance3 = getDistance(trig3, echo3);
  Serial.print("Distance3 (Front): ");
  Serial.print(distance3);
  Serial.println(" cm");
  delay(100);

  if (distance1 < 8) {
    Serial.println("RIGHT SIDE CRACK DETECTED");
    stopMotors();
    activateAlarm();
    if (!rightCrackAlerted && canSendSMS()) {
      sendSMS("ALERT: Right side crack detected by robot!");
      rightCrackAlerted = true;
      leftCrackAlerted = false;
      frontObstacleAlerted = false;
    }
    delay(1000);
  } else if (distance2 < 8) {
    Serial.println("LEFT SIDE CRACK DETECTED");
    stopMotors();
    activateAlarm();
    if (!leftCrackAlerted && canSendSMS()) {
      sendSMS("ALERT: Left side crack detected by robot!");
      leftCrackAlerted = true;
      rightCrackAlerted = false;
      frontObstacleAlerted = false;
    }
    delay(1000);
  } else if (distance3 < 8) {
    Serial.println("FRONT SIDE OBSTACLE DETECTED");
    stopMotors();
    activateAlarm();
    if (!frontObstacleAlerted && canSendSMS()) {
      sendSMS("ALERT: Front obstacle detected by robot!");
      frontObstacleAlerted = true;
      rightCrackAlerted = false;
      leftCrackAlerted = false;
    }
    delay(1000);
  } else {
    deactivateAlarm();
    moveForward();
    rightCrackAlerted = false;
    leftCrackAlerted = false;
    frontObstacleAlerted = false;
  }

  Serial.println("---------------------");
  delay(200);

  while (gsmSerial.available()) {
    Serial.write(gsmSerial.read());
  }
}

int getDistance(int trigPin, int echoPin) {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(5);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(20);
  digitalWrite(trigPin, LOW);
  long duration = pulseIn(echoPin, HIGH);
  int distance = duration * 0.034 / 2;
  return distance;
}

void stopMotors() {
  digitalWrite(Motor_pin1, LOW);
  digitalWrite(Motor_pin2, LOW);
  digitalWrite(Motor_pin3, LOW);
  digitalWrite(Motor_pin4, LOW);
}

void moveForward() {
  digitalWrite(Motor_pin1, HIGH);
  digitalWrite(Motor_pin2, LOW);
  digitalWrite(Motor_pin3, HIGH);
  digitalWrite(Motor_pin4, LOW);
}

void activateAlarm() {
  digitalWrite(Buzzer, HIGH);
}

void deactivateAlarm() {
  digitalWrite(Buzzer, LOW);
}

void initGSM() {
  Serial.println("Initializing GSM Module...");
  gsmSerial.println("AT");
  delay(1000);
  gsmSerial.println("AT+CMGF=1");
  delay(1000);
  Serial.println("GSM Module Ready");
}

void sendSMS(String message) {
  Serial.println("Sending SMS: " + message);
  gsmSerial.println("AT+CMGS=\"" + PHONE_NUMBER + "\"");
  delay(1000);
  gsmSerial.print(message);
  delay(100);
  gsmSerial.write(26);
  delay(1000);
  lastSMSTime = millis();
  Serial.println("SMS Sent");
}

bool canSendSMS() {
  return (millis() - lastSMSTime > SMS_COOLDOWN);
}
