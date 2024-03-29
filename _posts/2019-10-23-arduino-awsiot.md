---
layout: post
author: Tanveer jan
title: Using AWS IoT with Arduino MKR
date: 2019-10-23
thumbnail: /assets/img/posts/Arduino.jpg
tags: arduino awsiot
summary: Temperature data in AWS-IoT dashboard
---
This application transmits temperature and humidity from arduino to AWS IoT through wireless connection(WiFi). AWS IoT then store the record into DynamoDB.

```c
#include <ArduinoBearSSL.h>
#include <ArduinoECCX08.h>
#include <ArduinoMqttClient.h>
#include <WiFiNINA.h> // change to #include <WiFi101.h> for MKR1000
#include <DHT.h>
#include <BH1750.h>
#include <Wire.h>
#include "arduino_secrets.h"
BH1750 lightMeter(0x23);
float lux;

/////// Enter your sensitive data in arduino_secrets.h
const char ssid[]        = SECRET_SSID;
const char pass[]        = SECRET_PASS;
const char broker[]      = SECRET_BROKER;
const char* certificate  = SECRET_CERTIFICATE;
const int LCD_COLS = 20;
const int LCD_ROWS = 4;
WiFiClient    wifiClient;            // Used for the TCP socket connection
BearSSLClient sslClient(wifiClient); // Used for SSL/TLS connection, integrates with ECC508
MqttClient    mqttClient(sslClient);

#define DHTPIN A5     // what pin we're connected to
#define DHTTYPE DHT22   // DHT 22  (AM2302)
DHT dht(DHTPIN, DHTTYPE);

unsigned long lastMillis = 0;

int lightVal;
float temperatureVal;
void setup() {
  Serial.begin(115200);
  dht.begin();
  Wire.begin();
  lightMeter.begin();
  while (!Serial);

  if (!ECCX08.begin()) {
    Serial.println("No ECCX08 present!");
    while (1);
  }

  // Set a callback to get the current time
  // used to validate the servers certificate
  ArduinoBearSSL.onGetTime(getTime);

  // Set the ECCX08 slot to use for the private key
  // and the accompanying public certificate for it
  sslClient.setEccSlot(0, certificate);

  // Optional, set the client id used for MQTT,
  // each device that is connected to the broker
  // must have a unique client id. The MQTTClient will generate
  // a client id for you based on the millis() value if not set
  //
  // mqttClient.setId("clientId");

  // Set the message callback, this function is
  // called when the MQTTClient receives a message
  mqttClient.onMessage(onMessageReceived);
}

void loop() {

  if (WiFi.status() != WL_CONNECTED) {
    connectWiFi();
  }

  if (!mqttClient.connected()) {
    // MQTT client is disconnected, connect
    connectMQTT();
  }

  // poll for new MQTT messages and send keep alives
  mqttClient.poll();

  // publish a message roughly every 5 seconds.
  if (millis() - lastMillis > 5000) {
    lastMillis = millis();
    int lightVal = analogRead(A4);
    if(lightVal > 0)
    {
     publishMessage();
    }

  }
}

unsigned long getTime() {
  // get the current time from the WiFi module  
  return WiFi.getTime();
}

void connectWiFi() {
  Serial.print("Attempting to connect to SSID: ");
  Serial.print(ssid);
  Serial.print(" ");

  while (WiFi.begin(ssid, pass) != WL_CONNECTED) {
    // failed, retry
    Serial.print(".");
    delay(5000);
  }
  Serial.println();

  Serial.println("You're connected to the network");
  Serial.println();
}

void connectMQTT() {
  Serial.print("Attempting to MQTT broker: ");
  Serial.print(broker);
  Serial.println(" ");

  while (!mqttClient.connect(broker, 8883)) {
    // failed, retry
    Serial.print(".");
    delay(5000);
  }
  Serial.println();

  Serial.println("You're connected to the MQTT broker");
  Serial.println();

  // subscribe to a topic
  mqttClient.subscribe("arduino/incoming");
}

void publishMessage() {
  lux = lightMeter.readLightLevel();
  //lightVal = analogRead(A4);
  temperatureVal = dht.readTemperature();

  Serial.println("Publishing message");
  //Serial.println("Light:");
  //Serial.println(lightVal);
  Serial.println("temperature: ");
  Serial.println(temperatureVal);
  Serial.println("light : ");
  Serial.println(lux);

  // send message, the Print interface can be used to set the message contents
  mqttClient.beginMessage("arduino/outgoing");
  mqttClient.print("light:");
  mqttClient.print(lux);
  mqttClient.print(", Temp: ");
  mqttClient.print(temperatureVal);
  mqttClient.endMessage();
}

void onMessageReceived(int messageSize) {
  // we received a message, print out the topic and contents
  Serial.print("Received a message with topic '");
  Serial.print(mqttClient.messageTopic());
  Serial.print("', length ");
  Serial.print(messageSize);
  Serial.println(" bytes:");

  // use the Stream interface to print the contents
  while (mqttClient.available()) {
    Serial.print((char)mqttClient.read());
  }
  Serial.println();

  Serial.println();
}
```

Seperate tab should also be created and filled the credentials for wireless connection and certificates fro AWS-IoT

```c
// Fill in  your WiFi networks SSID and password
#define SECRET_SSID "SSID Here"
#define SECRET_PASS "Password Here"

// Fill in the hostname of your AWS IoT broker
#define SECRET_BROKER "endpoint address here"

// Fill in the boards public certificate
const char SECRET_CERTIFICATE[] = R"(
-----BEGIN CERTIFICATE-----
YOUR AWS-IoT Certificate here
-----END CERTIFICATE-----
)";
```