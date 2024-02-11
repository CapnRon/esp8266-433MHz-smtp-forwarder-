#include <ESP8266WiFi.h>
#include <ELECHOUSE_CC1101_SRC_DRV.h>
#include <ESP_Mail_Client.h>

#define WIFI_SSID     "SSID"
#define WIFI_PASS     "PASSWD"

// Add your email configuration here
#define SMTP_HOST     "smtp.gmail.com"
#define SMTP_PORT     esp_mail_smtp_port_587
#define AUTHOR_EMAIL  "your_email@gmail.com"
#define AUTHOR_PASSWORD  "your_email_password"
#define RECIPIENT_EMAIL "recipient_email@example.com"

#define SIGNAL_DEBOUNCE_SEC 3

WiFiClient wifiClient;

ELECHOUSE_CC1101 cc1101;

SMTPSession smtp;

uint32_t last_signal_value = 0;
uint32_t last_signal_timeout = 0;

// Uncomment the following line to enable debugging
// #define DEBUG_MESSAGES

void setup() {
  Serial.begin(115200);
  Serial.println();

  Serial.printf("Connecting to WiFi network %s ", WIFI_SSID);
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println(" connected.");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

  cc1101.Init();
  cc1101.setCCMode(1);
  cc1101.setModulation(0);
  cc1101.setMHZ(433.92);
  cc1101.setSyncMode(2);
  cc1101.setCrc(1);

  Serial.println("Rx Mode");

  // Set the callback function to get the sending results
  smtp.callback(smtpCallback);
}

void loop() {
  byte buffer[61] = {0};

  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("WiFi connection lost. Restarting.");
    delay(5000);
    ESP.restart();
  }

  handleReceivedSignal(buffer);

  // Print the email sending status every second
  static unsigned long lastEmailPrint = 0;
  if (millis() - lastEmailPrint >= 1000) {
    lastEmailPrint = millis();
    printEmailStatus();
  }
}

void handleReceivedSignal(byte* buffer) {
  if (millis() > last_signal_timeout) {
    last_signal_value = 0;
    last_signal_timeout = 0;
  }

  if (cc1101.CheckRxFifo(100)) {
    if (cc1101.CheckCRC()) {
      #ifdef DEBUG_MESSAGES
        Serial.print("Rssi: ");
        Serial.println(cc1101.getRssi());
        Serial.print("LQI: ");
        Serial.println(cc1101.getLqi());
      #endif

      int len = cc1101.ReceiveData(buffer);
      buffer[len] = '\0';
      #ifdef DEBUG_MESSAGES
        Serial.print("Received Signal: ");
        Serial.println((char *)buffer);
      #endif

      // Add email sending code here
      sendEmail((char *)buffer);

      last_signal_timeout = millis() + (SIGNAL_DEBOUNCE_SEC * 1000);
    }
  }
}

void sendEmail(char* message) {
  Session_Config config;

  config.server.host_name = SMTP_HOST;
  config.server.port = SMTP_PORT;
  config.login.email = AUTHOR_EMAIL;
  config.login.password = AUTHOR_PASSWORD;
  config.login.user_domain = F("127.0.0.1");

  if (!smtp.connect(&config)) {
    #ifdef DEBUG_MESSAGES
      Serial.println("Connection error to SMTP server.");
    #endif
    return;
  }

  SMTP_Message email;
  email.sender.name = F("Arduino Device");
  email.sender.email = AUTHOR_EMAIL;
  email.subject = "Arduino Message Received";
  email.addRecipient(RECIPIENT_EMAIL, RECIPIENT_EMAIL);
  email.text.content = message;

  if (!MailClient.sendMail(&smtp, &email)) {
    #ifdef DEBUG_MESSAGES
      Serial.println("Failed to send email.");
    #endif
  }
}

void smtpCallback(SMTP_Status status) {
  // Handle email sending callback here if needed
}

void printEmailStatus() {
  #ifdef DEBUG_MESSAGES
    if (smtp.connected()) {
      Serial.println("Email Connection Status: Connected");
    } else {
      Serial.println("Email Connection Status: Disconnected");
    }
  #endif
}
