# water_quality_IoT
This as the code for the IoT components used to check the quality of the water using ph sensor, turbidity sensor, tds and temperature sensor it will show the output in the serial monitor only 
#define TDS_PIN A0
#define TURBIDITY_PIN A1
#define PH_PIN A2
#define TEMP_PIN A3
#define BUZZER_PIN 3

#define VCC 5.0
#define ADC_MAX 1023.0

void setup() {
  Serial.begin(9600);
}

// TDS Sensor
float readTDS() {
  int raw = analogRead(TDS_PIN);
  float voltage = (raw / ADC_MAX) * VCC;
  return voltage * 3333.33;  
}

// Turbidity Sensor (Fixed)
float readTurbidity() {
  int raw = analogRead(TURBIDITY_PIN);
  float voltage = (raw * VCC) / ADC_MAX;

  float turbidity = map(voltage * 100, 250, 420, 100, 0);
  turbidity = constrain(turbidity, 0, 1000);
  return turbidity / 100.0;
}

// pH Sensor 
float readPH() {
  int raw = analogRead(PH_PIN);
  float voltage = (raw / ADC_MAX) * VCC;
  return 7 + ((3.15 - voltage) / 0.18);
}

// Temperature Sensor 
float readTemperature() {
  int raw = analogRead(TEMP_PIN);
  float voltage = (raw * VCC) / ADC_MAX;
  float temperature = voltage * 100.0;
  return temperature * (27.5 / 296.19);  // Scale down
}

void loop() {
  float tds = readTDS();
  float turbidity = readTurbidity();
  float ph = readPH();
  float temp = readTemperature();

  Serial.print("pH: "); Serial.print(ph, 2);
  Serial.print(" | Temp: "); Serial.print(temp, 2); Serial.print(" °C");
  Serial.print(" | Turbidity: "); Serial.print(turbidity, 2); Serial.print(" NTU");
  Serial.print(" | TDS: "); Serial.print(tds, 2); Serial.println(" mg/L");

  bool isPHSafe = (ph >= 6.5 && ph <= 8.5);
  bool isTDSSafe = (tds >= 0 && tds <= 500);
  bool isTurbiditySafe = (turbidity >= 0 && turbidity <= 1.0);
  bool isTempSafe = (temp >= 5.0 && temp <= 50.0);

  if (isPHSafe && isTDSSafe && isTurbiditySafe && isTempSafe) {
    Serial.println("Water is safe!\n");
    digitalWrite(BUZZER_PIN, LOW);
      delay(500);
  } else {
    Serial.println("Water is not safe!\n");
    digitalWrite(BUZZER_PIN, HIGH);
      delay(500);
  }

  delay(3000);
}
