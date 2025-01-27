# Temperatur_Based_Fan_Control


https://www.tinkercad.com/things/cGDQEZ79yh1-magnificent-fyyran-snicket?sharecode=ZduSj3VRiqwB0KlDq_qEvHirLPMvDMXMiErmi55erl8






![image](https://github.com/user-attachments/assets/65c80c01-9edb-470c-a558-2f7c40607c3c)

#include <EEPROM.h>
#include <LiquidCrystal.h>
LiquidCrystal lcd(2, 3, 4, 5, 6, 7);

#define TempSensorPin A0 // Pin for LM35 temperature sensor

#define bt_set  A3
#define bt_up   A4
#define bt_down A5

#define fan 9
#define buzzer 13

float setL_temp = 20.5; // Lower set temperature
float setH_temp = 60.5; // Higher set temperature
float temperature = 0;

int duty_cycle;
int Set = 0, flag = 0, flash = 0;

void setup() {
  pinMode(TempSensorPin, INPUT);

  pinMode(bt_set, INPUT_PULLUP);
  pinMode(bt_up, INPUT_PULLUP);
  pinMode(bt_down, INPUT_PULLUP);

  pinMode(fan, OUTPUT);
  pinMode(buzzer, OUTPUT);

  lcd.begin(16, 2); // Initialize LCD (16 columns and 2 rows)
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Temperature");
  lcd.setCursor(0, 1);
  lcd.print("Fan Control");


  // Load saved temperature settings from EEPROM
  if (EEPROM.read(0) != 0) {
    EEPROM.get(10, setL_temp);
    EEPROM.get(15, setH_temp);
  }

  delay(2000); // Display welcome message for 2 seconds
  lcd.clear();
}

void loop() {
  int thetakendata = analogRead(TempSensorPin);
  float resistance = (thetakendata / 1024.0) * 5000; // Converts the reading to millivolts
  temperature = (resistance - 500) * 0.1; // Converts resistance to temperature

  int value1 = temperature * 10;
  int value2 = setL_temp * 10;
  int value3 = setH_temp * 10;

  duty_cycle = map(value1, value2, value3, 0, 100);
  if (duty_cycle > 100) duty_cycle = 100;
  if (duty_cycle < 0) duty_cycle = 0;

  if (temperature < setL_temp) {
    digitalWrite(fan, LOW);
  } else {
    analogWrite(fan, (duty_cycle * 2) + 55);
  }

  if (temperature > setH_temp) {
    digitalWrite(buzzer, HIGH);
    delay(100);
  }

  handleButtons();
  displayTemperature();
  digitalWrite(buzzer, LOW); 
}

void handleButtons() {
  if (digitalRead(bt_set) == 0) {
    digitalWrite(buzzer, HIGH);
    if (flag == 0) {
      flag = 1;
      Set++;
      if (Set > 2) Set = 0;
      delay(200);
    }
  } else {
    flag = 0;
  }

  if (digitalRead(bt_up) == 0) {
    digitalWrite(buzzer, HIGH);
    if (Set == 1) {
      setL_temp += 0.1;
      EEPROM.put(10, setL_temp);
    }
    if (Set == 2) {
      setH_temp += 0.1;
      EEPROM.put(15, setH_temp);
    }
    delay(10);
  }

  if (digitalRead(bt_down) == 0) {
    digitalWrite(buzzer, HIGH);
    if (Set == 1) {
      setL_temp -= 0.1;
      EEPROM.put(10, setL_temp);
    }
    if (Set == 2) {
      setH_temp -= 0.1;
      EEPROM.put(15, setH_temp);
    }
    delay(10);
  }
}

void displayTemperature() {
  lcd.clear();
  if (Set == 0) {
    lcd.setCursor(0, 0);
    lcd.print("Temp: ");
    lcd.print(temperature, 1);
    lcd.write(223); // Degree symbol
    lcd.print("C   ");

    lcd.setCursor(0, 1);
    lcd.print("Fan Speed:");
    if (duty_cycle < 100) lcd.print(" ");
    lcd.print(duty_cycle);
    lcd.print("%   ");
  } else {
    lcd.setCursor(0, 0);
    lcd.print("Setting Mode");
    lcd.setCursor(0, 1);
    lcd.print("L:");
    if (Set == 1 && flash) {
      lcd.print("    ");
    } else {
      lcd.print(setL_temp, 1);
    }
    lcd.print("C  H:");
    if (Set == 2 && flash) {
      lcd.print("    ");
    } else {
      lcd.print(setH_temp, 1);
    }
    lcd.print("C  ");
    flash = !flash; // Toggle flash state
  }
  delay(500); // Update display every 500 ms
}

