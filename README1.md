# HACKATHONjntu
Accident prevention detection and reporting system 
#include <SoftwareSerial.h>
SoftwareSerial mySerial(9, 10);
#define x A3
#define y A4
#define z A5
int xsample = 0;
int ysample = 0;
int zsample = 0;
#define samples 10
#define minVal -50
#define MaxVal 50
int i = 0, k = 0;
#define Relay A1
#define buzzer 13
#define LED1 2
#define LED2 3
//alcohol sensor
#define MQ3 A0
#define Buzzer 13
#define LED3 4
#define LED4 5
#define Thres_Val 460
const int trigPin = 6;
const int echoPin = 12;
int sensorInput;
int tempsensePin = A2;
int temp;
int LED5 = 7;
int LED6 = 8 ;
int threshold_temperature2 = 100;
int threshold_temperature1 = 30;
int value;
static const int sensorPin = 11;                    // sensor input pin
int SensorStatePrevious = LOW;                      // previousstate of the sensor
unsigned long minSensorDuration = 3000; // Time we wait before  the sensor active as long
unsigned long minSensorDuration2 = 6000;
unsigned long SensorLongMillis;                // Time in ms when the sensor was active
bool SensorStateLongTime = false;                  // True if it is a long active
const int intervalSensor = 50;                      // Time between two readings sensor state
unsigned long previousSensorMillis;                 // Timestamp of the latest reading
unsigned long SensorOutDuration;                  // Time the sensor is active in ms
unsigned long currentMillis;          // Variabele to store the number of milleseconds since the Arduino has started
long duration;
int distance;
void setup() {
  mySerial.begin(9600);   // Setting the baud rate of GSM Module
  Serial.begin(9600);    // Setting the baud rate of Serial Monitor (Arduino)
  delay(100);
  Serial.begin(9600);
  Serial.println("Initializing....");
  Serial.println("Initialized Successfully");
  for (int i = 0; i < samples; i++)
  {
    xsample += analogRead(x);
    ysample += analogRead(y);
    zsample += analogRead(z);
  }

  xsample /= samples;
  ysample /= samples;
  zsample /= samples;

  Serial.println(xsample);
  Serial.println(ysample);
  Serial.println(zsample);
  delay(1000);
  pinMode(trigPin, OUTPUT); // Sets the trigPin as an Output
  pinMode(echoPin, INPUT); // Sets the echoPin as an Input
  pinMode(Relay, OUTPUT); // Sets the Motor as an Output
  Serial.begin(9600);                 // Initialise the serial monitor
  pinMode(sensorPin, INPUT);          // set sensorPin as input
  Serial.println("Press button");
  pinMode(Relay, OUTPUT);
  pinMode(buzzer, OUTPUT);
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
  pinMode(LED3, OUTPUT);
  pinMode(MQ3, INPUT);
  pinMode(buzzer, OUTPUT);
  pinMode(LED4, OUTPUT);
  pinMode(LED5, OUTPUT);
  pinMode(LED6, OUTPUT);
  pinMode(tempsensePin, INPUT);
  pinMode(sensorInput, INPUT);
  pinMode (buzzer, OUTPUT);
  // Function for reading the sensor state
}
void readSensorState() {
  digitalWrite(LED1, HIGH);
  // If the difference in time between the previous reading is larger than intervalsensor
  if (currentMillis - previousSensorMillis > intervalSensor) {
    // Read the digital value of the sensor (LOW/HIGH)
    int SensorState = digitalRead(sensorPin);
    // If the button has been active AND
    // If the sensor wasn't activated before AND
    // IF there was not already a measurement running to determine how long the sensor has been activated
    if (SensorState == LOW && SensorStatePrevious == HIGH && !SensorStateLongTime) {
      SensorLongMillis = currentMillis;
      SensorStatePrevious = LOW;
      //digitalWrite(buzzer,HIGH);
      Serial.println("Button pressed");
    }
    // Calculate how long the sensor has been activated
    SensorOutDuration = currentMillis - SensorLongMillis;
    // If the button is active AND
    // If there is no measurement running to determine how long the sensor is active AND
    // If the time the sensor has been activated is larger or equal to the time needed for a long active
    if (SensorState == LOW && !SensorStateLongTime && SensorOutDuration >= minSensorDuration) {
      SensorStateLongTime = true;
      digitalWrite(Relay, HIGH);
      digitalWrite(LED2, HIGH);


      Serial.println("Button long pressed");
    }
    if (SensorState == LOW && SensorStateLongTime && SensorOutDuration >= minSensorDuration2) {
      SensorStateLongTime = true;
      digitalWrite(buzzer, HIGH);
      digitalWrite(LED2, HIGH);
      Serial.println("Button long pressed");
    }
    // If the sensor is released AND
    // If the sensor was activated before
    if (SensorState == HIGH && SensorStatePrevious == LOW) {
      SensorStatePrevious = HIGH;
      SensorStateLongTime = false;
      digitalWrite(Relay, LOW);
      digitalWrite(buzzer, LOW);
      digitalWrite(LED1, HIGH);
      digitalWrite(LED2, LOW);
      Serial.println("Button released");
    }
    // store the current timestamp in previousSensorMillis
    previousSensorMillis = currentMillis;
  }
}
void loop() {
  temp = analogRead(A2);
  temp = temp * 0.48828125;
  currentMillis = millis();    // store the current time
  readSensorState();           // read the sensor state // reads the analog value from MQ3
  value = analogRead(MQ3);
  // sends the value to UART for debugging
  Serial.println(value);
  if ( value = 0)  //if alcohol is detected
  {
    digitalWrite ( LED3, LOW);    // turns the LED on
    digitalWrite(Buzzer, HIGH);
  }
  if ( value > Thres_Val )   //if alcohol is detected
  { digitalWrite ( LED4 , LOW);
    digitalWrite ( LED3, HIGH);       // turns the LED on
    digitalWrite(Buzzer, LOW);
  } else {
    digitalWrite(LED3, LOW);      //  turns the LED off
    digitalWrite(LED6, HIGH);
    digitalWrite(Buzzer, HIGH);
  }
  delay (500);            //  a 500ms delay before reading is taken again
  if (temp > threshold_temperature1)
  {
    Serial.print("Temperature: ");
    digitalWrite(LED5, HIGH);
    digitalWrite(buzzer, LOW);
  }
  if (temp > threshold_temperature2)
  {
    Serial.print("Temperature: ");
    digitalWrite(LED6, HIGH);
    digitalWrite(LED5, LOW);
    digitalWrite(buzzer, HIGH);
    digitalWrite(Relay, HIGH);
  }
  else
  {
    digitalWrite(LED5, HIGH);
    digitalWrite(LED6, LOW);
    digitalWrite(buzzer, LOW);
    Serial.print("Temp is fine");
  } digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  // Sets the trigPin on HIGH state for 10 micro seconds
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  // Reads the echoPin, returns the sound wave travel time in microseconds
  duration = pulseIn(echoPin, HIGH);
  // Calculating the distance
  distance = duration * 0.034 / 2;
  // Prints the distance on the Serial Monitor
  Serial.print("Distance: ");
  Serial.println(distance);
  if (distance < 10) {
    digitalWrite(Relay, HIGH);
  }
  else
  {
    digitalWrite(Relay, HIGH);
  }
  int value1 = analogRead(x);
  int value2 = analogRead(y);
  int value3 = analogRead(z);
  int xValue = xsample - value1;
  int yValue = ysample - value2;
  int zValue = zsample - value3;
  Serial.print("x=");
  Serial.println(xValue);
  Serial.print("y=");
  Serial.println(yValue);
  Serial.print("z=");
  Serial.println(zValue);
}
void gpsEvent()
{
  gpsString = "";
  while (1)
  {
    while (gps.available() > 0)          //Serial incoming data from GPS
    {
      char inChar = (char)gps.read();
      gpsString += inChar;                   //store incoming data from GPS to temparary string str[]
      i++;
      // Serial.print(inChar);
      if (i < 7)
      {
        if (gpsString[i - 1] != test[i - 1])    //check for right string
        {
          i = 0;
          gpsString = "";
        }   }
      if (inChar == '\r')
      {   if (i > 60)
        {  gps_status = 1;
          break; }
        else
        {
          i = 0; } }   }
    if (gps_status)
      break;
  }}
void get_gps()
{
  gps_status = 0;
  int x = 0;
  while (gps_status == 0)
  {
    gpsEvent();
    int str_lenth = i;
    coordinate2dec();
    i = 0; x = 0;
    str_lenth = 0;
  }
}void coordinate2dec()
{
  String lat_degree = "";
  for (i = 20; i <= 21; i++)
    lat_degree += gpsString[i];

  String lat_minut = "";
  for (i = 22; i <= 28; i++)
    lat_minut += gpsString[i];
 String log_degree = "";
  for (i = 32; i <= 34; i++)
    log_degree += gpsString[i];
  String log_minut = "";
  for (i = 35; i <= 41; i++)
    log_minut += gpsString[i];
  Speed = "";
  for (i = 45; i < 48; i++)   //extract longitude from string
    Speed += gpsString[i];

  float minut = lat_minut.toFloat();
  minut = minut / 60;
  float degree = lat_degree.toFloat();
  latitude = degree + minut;

  minut = log_minut.toFloat();
  minut = minut / 60;
  degree = log_degree.toFloat();
  logitude = degree + minut;
}
void Send(){
  Serial1.println("AT");
  delay(500);
  serialPrint();
  Serial1.println("AT+CMGF=1");
  delay(500);
  serialPrint();
  Serial1.print("AT+CMGS=");
  Serial1.print('"');
  Serial1.print("+9779800000000"); //mobile no. for SMS alert
  Serial1.println('"');
  delay(500);
  serialPrint();
  Serial1.print("Latitude:");
  Serial1.println(latitude);
  delay(500);
  serialPrint();
  Serial1.print(" longitude:");
  Serial1.println(logitude);
  delay(500);
  serialPrint();
  Serial1.print(" Speed:");
  Serial1.print(Speed);
  Serial1.println("Knots");
  delay(500);
  serialPrint();
  Serial1.print("http://maps.google.com/maps?&z=15&mrt=yp&t=k&q=");
  Serial1.print(latitude, 6);
  Serial1.print("+");              //28.612953, 77.231545   //28.612953,77.2293563
  Serial1.print(logitude, 6);
  Serial1.write(26);
  delay(2000);
  serialPrint();
}void serialPrint()
{  while (Serial1.available() > 0)
  {  Serial.print(Serial1.read());
  }}
