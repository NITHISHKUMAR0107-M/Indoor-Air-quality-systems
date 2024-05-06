# Indoor-Air-quality-systems![Indoor air qulity system](https://github.com/NITHISHKUMAR0107-M/Indoor-Air-quality-systems/assets/127758446/a466dda9-e558-4ef3-b0ff-a549efe31e5b)

Project![Air quality level dashboard](https://github.com/NITHISHKUMAR0107-M/Indoor-Air-quality-systems/assets/127758446/ba283075-68e2-405a-8ce1-b9bb3726abe2)
[air quality abstract.txt](https://github.com/NITHISHKUMAR0107-M/Indoor-Air-quality-systems/files/15223756/air.quality.abstract.txt)

IoT BASED INDOOR AIR QUALITY MONITORING SYSTEM


Abstract:
Air pollution is a major concern across the globe due to its negative impact on public health and the environment. Particulate matter (PM) is one of the primary pollutants that contribute to air pollution. PM2.5, which refers to particles with a diameter less than or equal to 2.5 microns, is particularly harmful to human health as it can penetrate deep into the lungs and bloodstream. Hence, it is crucial to monitor the levels of PM2.5 in the air. This concept presents the design and implementation of a PM2.5 sensor using a NodeMCU microcontroller and a laser sensor. The sensor measures the concentration of PM1, PM2.5, and PM10 particles in the air and transmits the data to the Thingsboard cloud platform via the MQTT protocol. The collected data is then visualized in a beautiful dashboard, which enables users to monitor the air quality in real-time. The design and implementation of the PM2.5 sensor involve several steps, including the calibration of the laser sensor, the configuration of the NodeMCU microcontroller, and the integration of the sensor with the Thingsboard platform. The paper provides a detailed description of each step and the challenges encountered during the implementation process. The results of the implementation show that the PM2.5 sensor can accurately measure the concentration of PM1, PM2.5, and PM10 particles in the air. The data transmitted to the Thingsboard platform is visualized in a beautiful dashboard that provides real-time monitoring of air quality. The dashboard includes several features, such as historical data analysis, alerts, and notifications, which enable users to take appropriate actions in cases of high pollution levels. The implementation of the PM2.5 sensor using a NodeMCU microcontroller and a laser sensor, integrated with the Thingsboard cloud platform, provides an effective solution for monitoring air quality in real-time. The beautiful dashboard provides a user-friendly interface for visualizing the collected data and taking appropriate actions to mitigate the effects of air pollution.

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
