#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <EEPROM.h>
#include <SoftwareSerial.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);
SoftwareSerial esp(10, 11); // RX, TX
const uint8_t pulse_in = 2; // interrupt pin

const int BTN1 = 5;  // OK / ENTER
const int BTN2 = 6;  // RESET
const int BTN3 = 7;  // SELECT

volatile unsigned long pulseCount = 0;
volatile unsigned long total_pulses_all = 0;
float total_units = 0.0;
float unit_price = 5.5;
unsigned long lastSaveMillis = 0, lastDisplayMillis = 0;
const unsigned long DISPLAY_INTERVAL = 500;
const unsigned long SAVE_INTERVAL = 20000;
const unsigned long ISR_DEBOUNCE_US = 5000;
volatile unsigned long lastPulseMicros = 0;

float total_cost = 0;

String ssid = "energy";
String password = "12345678";
String ipAddress = "";

// -------------------------------
bool readButton(int pin) { return digitalRead(pin) == LOW; }
void waitRelease(int pin) { while (digitalRead(pin) == LOW); delay(200); }

void setupButtons() {
  pinMode(BTN1, INPUT_PULLUP);
  pinMode(BTN2, INPUT_PULLUP);
  pinMode(BTN3, INPUT_PULLUP);
}
// -------------------------------

// ? Pulse Interrupt
void countPulse() {
  unsigned long now = micros();
  if ((now - lastPulseMicros) >= ISR_DEBOUNCE_US) {
    pulseCount++;
    total_pulses_all++;
    lastPulseMicros = now;
  }
}

// ======= PRICE SELECTION ========
void priceSelection() {
  float prices[] = {5, 5.5, 6, 6.5, 7, 7.5};
  int index = 0;

  for (int i = 0; i < 6; i++) {
    if (prices[i] == unit_price) {
      index = i;
      break;
    }
  }

  lcd.clear();
  lcd.setCursor(0, 0); lcd.print("Select Price:");
  lcd.setCursor(0, 1); lcd.print("Rs "); lcd.print(prices[index], 1);

  while (true) {
    if (readButton(BTN3)) {
      index = (index + 1) % 6;
      lcd.setCursor(0, 1);
      lcd.print("Rs "); lcd.print(prices[index], 1); lcd.print("   ");
      waitRelease(BTN3);
    }
    if (readButton(BTN1)) {
      unit_price = prices[index];
      EEPROM.put(sizeof(total_units) + sizeof(total_pulses_all), unit_price);
      lcd.clear();
      lcd.setCursor(0, 0); lcd.print("Selected:");
      lcd.setCursor(0, 1); lcd.print("Rs "); lcd.print(unit_price, 1);
      delay(1000);
      waitRelease(BTN1);
      break;
    }
  }
}

// ======= RESET ROUTINE ========
void resetData() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Reset & Select?");
  lcd.setCursor(0, 1);
  lcd.print("YES=D5  NO=D6");

  unsigned long start = millis();
  while (millis() - start < 10000) {
    if (readButton(BTN1)) {
      lcd.clear();
      lcd.print("Resetting...");
      total_units = 0;
      total_pulses_all = 0;
      EEPROM.put(0, total_units);
      EEPROM.put(sizeof(total_units), total_pulses_all);
      delay(1000);
      waitRelease(BTN1);
      priceSelection();
      lcd.clear();
      lcd.print("New price set!");
      delay(1000);
      lcd.clear();
      return;
    }
    if (readButton(BTN2)) {
      lcd.clear();
      lcd.print("Cancelled");
      delay(1000);
      waitRelease(BTN2);
      return;
    }
  }
  lcd.clear();
}

// ======= ESP COMMAND FUNCTION =======
void sendToESP(String cmd, const char* ack, int t) {
  esp.println(cmd);
  long int time = millis();
  while ((time + t) > millis()) {
    while (esp.available()) {
      String res = esp.readString();
      if (res.indexOf(ack) != -1) return;
    }
  }
}

// ======= WIFI SETUP =======
void setupWiFi() {
  Serial.println("Connecting to WiFi...");
  sendToESP("AT", "OK", 1000);
  sendToESP("AT+CWMODE=1", "OK", 1000);
  sendToESP("AT+CWJAP=\"" + ssid + "\",\"" + password + "\"", "OK", 6000);
  sendToESP("AT+CIFSR", "OK", 2000);

  esp.println("AT+CIFSR");
  delay(2000);
  while (esp.available()) {
    String res = esp.readString();
    Serial.println(res);
    int idx = res.indexOf("STAIP,\"");
    if (idx != -1) {
      int start = idx + 7, end = res.indexOf("\"", start);
      ipAddress = res.substring(start, end);
      Serial.print("ESP-01 IP Address: ");
      Serial.println(ipAddress);
      break;
    }
  }
  sendToESP("AT+CIPMUX=1", "OK", 1000);
  sendToESP("AT+CIPSERVER=1,80", "OK", 1000);
  Serial.println("Web server started!");
}

// ======= WEBPAGE =======
void sendWebpage() {
  if (esp.available() && esp.find("+IPD,")) {
    delay(50);
    String request = esp.readStringUntil('\r');
    esp.readStringUntil('\n');

    String webpage = "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n";
    webpage += "<html><head><title>Energy Meter</title></head><body>";
    webpage += "<h2>Smart Energy Meter</h2>";
    webpage += "<p><b>Selected Price:</b> " + String(unit_price, 1) + "</p>";
    webpage += "<p><b>Total Units:</b> " + String(total_units, 1) + "</p>";
    webpage += "<p><b>Total Cost:</b> Rs. " + String(total_cost, 1) + "</p>";
    //webpage += "<p><b>Selected Price:</b> Rs. " + String(unit_price, 1) + "</p>";
    webpage += "<hr><p>Refresh page to update data.</p>";
    webpage += "</body></html>";

    String sendCmd = "AT+CIPSEND=0," + String(webpage.length());
    sendToESP(sendCmd, ">", 1000);
    esp.print(webpage);
    delay(100);
    esp.println("AT+CIPCLOSE=0");
  }
}

// ======= SETUP =======
void setup() {
  Serial.begin(9600);
  esp.begin(9600);
  lcd.begin(); lcd.backlight();
  setupButtons();
  pinMode(pulse_in, INPUT);

  EEPROM.get(0, total_units);
  EEPROM.get(sizeof(total_units), total_pulses_all);
  EEPROM.get(sizeof(total_units) + sizeof(total_pulses_all), unit_price);

  lcd.clear();
  lcd.setCursor(0,0); lcd.print("Prev Price Rs:");
  lcd.setCursor(0,1); lcd.print(unit_price,1);
  delay(2000);

  lcd.clear();
  lcd.print("New price? ");
  lcd.setCursor(0,1); lcd.print("YES=D7  NO=D5");

  unsigned long start = millis();
  bool selected = false;
  while ((millis() - start) < 10000) {
    if (readButton(BTN3)) {
      waitRelease(BTN3);
      priceSelection(); 
      selected = true;
      break;
    }
    if (readButton(BTN1)) {
      waitRelease(BTN1);
      selected = true;
      break;
    }
  }
  if (!selected) {
    lcd.clear(); lcd.print("Keep old price");
    delay(1000);
  }

  attachInterrupt(digitalPinToInterrupt(pulse_in), countPulse, RISING);

  lcd.clear();
  lcd.setCursor(2,0); lcd.print("Energy Meter");
  lcd.setCursor(3,1); lcd.print("Initializing");

  setupWiFi();

  lcd.clear();
  lcd.setCursor(0,0); lcd.print("IP:");
  lcd.setCursor(0,1); lcd.print(ipAddress);
  delay(2000); lcd.clear();

  lastSaveMillis = millis();
  lastDisplayMillis = millis();
}

// ======= LOOP =======
void loop() {
  noInterrupts();
  unsigned long pulses = pulseCount;
  interrupts();

  // ? Every 32 pulses = 0.1 unit
  while (pulses >= 32) {
    pulses -= 32;
    noInterrupts();
    pulseCount = pulses;
    interrupts();
    total_units += 0.1;
  }

  unsigned long now = millis();

  if ((now - lastSaveMillis) >= SAVE_INTERVAL) {
    EEPROM.put(0, total_units);
    EEPROM.put(sizeof(total_units), total_pulses_all);
    EEPROM.put(sizeof(total_units) + sizeof(total_pulses_all), unit_price);
    lastSaveMillis = now;
  }

  total_cost = total_units * unit_price;

  if ((now - lastDisplayMillis) >= DISPLAY_INTERVAL) {
    lastDisplayMillis = now;
    lcd.setCursor(0,0);
    lcd.print("Unit:");
    lcd.print(total_units,1);
    lcd.print("   ");
    lcd.setCursor(12,0);
    lcd.print(pulseCount);
    lcd.print("  ");
    lcd.setCursor(0,1);
    lcd.print("Cost:");
    lcd.print(total_cost,1);
    lcd.print("  ");
    lcd.setCursor(11,1);
    lcd.print("(");
    lcd.setCursor(12,1);
    lcd.print(unit_price,1);
    lcd.setCursor(15,1);
    lcd.print(")");
  }

  if (readButton(BTN2)) {
    waitRelease(BTN2);
    resetData();
  }

  sendWebpage();
  delay(100);
}
