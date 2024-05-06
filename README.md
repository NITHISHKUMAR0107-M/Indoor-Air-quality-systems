# Indoor-Air-quality-systems
Project
#include <ESP8266WiFi.h> // For ESP8266
// OR
// #include <WiFi.h> // For ESP32
#include <PubSubClient.h> // MQTT client
#include "DFRobot_AirQualitySensor.h" // Air Quality Sensor

#define I2C_ADDRESS 0x19
DFRobot_AirQualitySensor particle(&Wire, I2C_ADDRESS);

// WiFi credentials
const char* ssid = "wifi_name"; //wife name
const char* password = "password"; //wifi password

// ThingsBoard server details
const char* thingsboard_server = "thingsboard.cloud";
const char* token = "Auth toked id";
const int port = 1883; // Standard MQTT port

WiFiClient wifiClient;
PubSubClient client(wifiClient);

void setup() {
  Serial.begin(115200);
  // Initialize WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("WiFi connected");
  client.setServer(thingsboard_server, port);

  // Initialize Air Quality Sensor
  while (!particle.begin()) {
    Serial.println("Sensor init failed, check wiring or I2C address");
    delay(1000);
  }
  Serial.println("Sensor initialized successfully");
}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Attempt to connect
    if (client.connect("AirQualitySensorClient", token, NULL)) {
      Serial.println("connected");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Read air quality data
  uint16_t pm1_0 = particle.gainParticleConcentration_ugm3(PARTICLE_PM1_0_ATMOSPHERE);
  uint16_t pm2_5 = particle.gainParticleConcentration_ugm3(PARTICLE_PM2_5_STANDARD);
  uint16_t pm10 = particle.gainParticleConcentration_ugm3(PARTICLE_PM10_STANDARD);

  // Prepare JSON payload
  String payload = "{\"pm1_0\":" + String(pm1_0) + ",\"pm2_5\":" + String(pm2_5) + ",\"pm10\":" + String(pm10) + "}";
  Serial.print("Sending payload: ");
  Serial.println(payload);

  // Send payload
  client.publish("v1/devices/me/telemetry", (char*) payload.c_str());

  delay(1000); // Wait for 10 seconds before sending again
}
