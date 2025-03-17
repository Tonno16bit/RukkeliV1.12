Below is a complete, step‐by‐step guide that summarizes everything we did—from setting up the Mosquitto broker on your Windows PC to configuring both ESP32 devices (one as a publisher and one as a subscriber with an SSD1306 OLED display using the u8g2 library). Follow these instructions in order:

Step 1: Set Up the Mosquitto MQTT Broker on Windows 10
1.1 Download and Install Mosquitto
Download: Visit the Mosquitto download page and download the Windows installer.

Install: Run the installer (typically installs to C:\Program Files\mosquitto\).

1.2 Configure the Broker
Create a configuration file (e.g., mosquitto.conf) and save it in C:\mosquitto\ (or another folder).

Use the following example configuration:

ini
listener 1883
allow_anonymous false
password_file C:\mosquitto\pwfile.txt
1.3 Create the Username/Password File
Open a Command Prompt and navigate to the Mosquitto installation folder:

cmd
cd "C:\Program Files\mosquitto\"
Run the following command to create the password file (replace your_username with your chosen username):

cmd
mosquitto_passwd -c C:\mosquitto\pwfile.txt your_username
When prompted, enter and confirm your password. This file is now used by Mosquitto to authenticate MQTT clients.

1.4 Set Up Port Forwarding and Firewall Configuration
Port Forwarding: Log in to your router’s administration interface and forward TCP port 1883 to your Windows PC’s internal IP address.

Firewall: Ensure your Windows Firewall allows inbound connections on port 1883.

1.5 Start the Mosquitto Broker
Open a Command Prompt and run:

cmd
"C:\Program Files\mosquitto\mosquitto.exe" -v -c "C:\mosquitto\mosquitto.conf"
The -v flag enables verbose logging so you can see connection messages, showing that the broker is up and running.

Step 2: Set Up the ESP32 Publisher (e.g., in Rome)
2.1 Prepare the Arduino IDE
Install the ESP32 board package (if not already done).

Install the PubSubClient library via the Library Manager.

2.2 Upload the Publisher Code
Use the following example code. Be sure to replace the placeholders (WiFi credentials, public IP/DDNS, MQTT username/password) with your values:

cpp
#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid         = "your_wifi_ssid";
const char* wifiPassword = "your_wifi_password";

const char* mqtt_server  = "YOUR_PUBLIC_IP_OR_DDNS";
const int   mqtt_port    = 1883;
const char* mqtt_user    = "your_username";
const char* mqtt_pass    = "your_password";

const char* topic        = "esp32/messages";

WiFiClient espClient;
PubSubClient client(espClient);

void setupWiFi() {
  WiFi.begin(ssid, wifiPassword);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }
}

void reconnectMQTT() {
  while (!client.connected()) {
    String clientId = "ESP32Pub-";
    clientId += String(random(0xffff), HEX);
    if (client.connect(clientId.c_str(), mqtt_user, mqtt_pass)) {
      client.publish(topic, "Publisher connected!");
    } else {
      delay(5000);
    }
  }
}

void setup() {
  setupWiFi();
  client.setServer(mqtt_server, mqtt_port);
}

void loop() {
  if (!client.connected()) {
    reconnectMQTT();
  }
  client.loop();

  // Publish message every 5 seconds
  static unsigned long lastMsg = 0;
  if (millis() - lastMsg > 5000) {
    lastMsg = millis();
    String message = "Hello from Publisher!";
    client.publish(topic, message.c_str());
  }
}
What Happens: This code connects your ESP32 to your WiFi network and MQTT broker, then publishes a message ("Hello from Publisher!") every 5 seconds on the topic "esp32/messages".

Step 3: Set Up the ESP32 Subscriber with an SSD1306 OLED Using u8g2 (e.g., in Venice)
3.1 Wiring and Hardware Setup
Display Resolution: Verify your display is 128×64.

I²C Wiring:

Connect SDA of the OLED to ESP32 pin 3.

Connect SCL of the OLED to ESP32 pin 4.

Connect VCC and GND appropriately (match the voltage specification of your module).

3.2 Prepare the Arduino IDE
Install the PubSubClient and U8g2 libraries via the Library Manager.

3.3 Upload the Subscriber Code
Use the following code, making sure you update your WiFi and MQTT details:

cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <Wire.h>
#include <U8g2lib.h>

// WiFi & MQTT settings
const char* ssid         = "your_wifi_ssid";
const char* wifiPassword = "your_wifi_password";
const char* mqtt_server  = "YOUR_PUBLIC_IP_OR_DDNS";
const int   mqtt_port    = 1883;
const char* mqtt_user    = "your_username";
const char* mqtt_pass    = "your_password";

const char* topic        = "esp32/messages";

WiFiClient espClient;
PubSubClient client(espClient);

// Full-buffer constructor for 128x64 OLED using hardware I2C with SCL on pin 4, SDA on pin 3.
U8G2_SSD1306_128X64_NONAME_F_HW_I2C u8g2(U8G2_R0, U8X8_PIN_NONE, 4, 3);

// MQTT callback: Called when a message is received.
void callback(char* topic, byte* payload, unsigned int length) {
  String message;
  for (unsigned int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("]: ");
  Serial.println(message);

  // Update the OLED display with the received message.
  u8g2.clearBuffer();                    
  u8g2.setFont(u8g2_font_ncenB08_tr);
  u8g2.drawStr(0, 20, "Received:");
  u8g2.drawStr(0, 40, message.c_str());
  u8g2.sendBuffer();
}

// Connect to WiFi
void setupWiFi() {
  Serial.print("Connecting to WiFi");
  WiFi.begin(ssid, wifiPassword);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.print("WiFi connected! IP: ");
  Serial.println(WiFi.localIP());
}

// Ensure the MQTT client is connected.
void reconnectMQTT() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    String clientId = "ESP32Sub-";
    clientId += String(random(0xffff), HEX);
    if (client.connect(clientId.c_str(), mqtt_user, mqtt_pass)) {
      Serial.println(" connected.");
      client.subscribe(topic);
    } else {
      Serial.print(" failed, rc=");
      Serial.print(client.state());
      Serial.println(" - retrying in 5 seconds.");
      delay(5000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  setupWiFi();
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
  
  // Initialize the full-buffer OLED display.
  u8g2.begin();
  delay(100);  // Allow some time for the display to initialize
  u8g2.clearBuffer();
  u8g2.setFont(u8g2_font_ncenB08_tr);
  u8g2.drawStr(0, 20, "Waiting for msg...");
  u8g2.sendBuffer();
}

void loop() {
  if (!client.connected()) {
    reconnectMQTT();
  }
  client.loop();
}
What Happens: The subscriber connects to your WiFi and broker. It subscribes to "esp32/messages" and—when it receives a message—the callback prints it to the Serial Monitor and shows it on the OLED display.

Step 4: Test the Whole System
Start Mosquitto Broker: Ensure your Windows PC is running the Mosquitto broker using the configuration you set up.

Deploy the Code:

Upload the publisher code to the ESP32 that you want to act as the publisher.

Upload the subscriber code to the ESP32 that is connected to the SSD1306 OLED.

Verify Functionality:

Open the Serial Monitor on both devices to see connection logs and MQTT status.

Observe the OLED (subscriber): It should initially display “Waiting for msg...” and then update with each incoming message (for example, “Hello from Publisher!” published every 5 seconds).
