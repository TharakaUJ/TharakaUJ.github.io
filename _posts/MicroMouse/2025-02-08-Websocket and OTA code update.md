# Enabling OTA and WebSockets for Micromouse

This article is based on my experience while trying to build a micromouse. In this article, I discuss how to enable OTA (over-the-air) programming and WebSocket to update variables.

You can find the documentation for Arduino OTA [here](https://docs.arduino.cc/arduino-cloud/features/ota-getting-started/).

## Setting Up WebSocket

You can set up the WebSocket as follows:

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <ArduinoWebsockets.h>

using namespace websockets;

WebsocketsServer webSocket;
WebsocketsClient client;

const char *ssid = "your_SSID";
const char *password = "your_PASSWORD";

// Function to initialize Wi-Fi
void setupWiFi()
{
    Serial.print("Connecting to Wi-Fi...");
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED)
    {
        delay(500);
        Serial.print(".");
    }
    Serial.println("\nConnected to Wi-Fi. IP: " + WiFi.localIP().toString());
}

void setupWireless()
{
    setupWiFi(); // Initialize Wi-Fi

    webSocket.listen(80); // Start WebSocket server
    Serial.println("WebSocket server started. Connect to ws://" + WiFi.localIP().toString() + ":80");
}

void loopWireless(void *parameter)
{
    for (;;)
    {
        // Infinite loop
        if (!client.available())
        {
            client = webSocket.accept();
            Serial.println("Client connected");
        }
        else
        {
            client.poll();
            auto message = client.readBlocking();
            handleWrite(message);
            Serial.println("Message received");
        }

        vTaskDelay(100 / portTICK_PERIOD_MS); // Non-blocking delay
    }
}
```

### Running WebSocket as a Task

Here, I used an infinite loop so that I could create a task for the loop, preventing it from being blocked by other tasks like reading sensors or turning. The setup function looks something like this:

```cpp
void setup() {
    // WiFi
    setupWireless();

    // Create a FreeRTOS task for the WiFi loop
    xTaskCreate(
        loopWireless,    // Task function
        "WiFi Task",     // Task name
        4096,            // Stack size (in bytes)
        NULL,            // Task parameters
        1,               // Priority
        NULL             // Task handle
    );
}
```

**Note:** I implemented the WebSocket so that it does not wait for new clients once one is connected. This means only one device can connect at a time in my code.

## Handling WebSocket and OTA Together

One challenge I faced was that when implementing both WebSocket and Arduino OTA, the WebSocket blocked the upload process. When we try to upload, the computer waits for the micromouse, but the micromouse does not accept the update.

The simplest way to avoid this issue was to close the WebSocket before uploading code. I added a parameter to WebSocket to toggle whether uploads are enabled or not.

### Implementation

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <ArduinoWebsockets.h>
#include <ArduinoOTA.h>
#include <ArduinoJson.h>
#include "variablesAndParameters.h"
#include "algorithm.h"

using namespace websockets;

const char *ssid = "your_SSID";
const char *password = "your_PASSWORD";

WebsocketsServer webSocket;
WebsocketsClient client;

bool upload = false;

// Send JSON data for `read` action
void handleRead()
{
    Serial.println("Sending data to client");
    DynamicJsonDocument doc(1024);
    doc["action"] = "read";
    doc["upload"] = upload;

    String response;
    serializeJson(doc, response);
    client.send(response);
}

// Handle incoming WebSocket messages
void handleWrite(const WebsocketsMessage &message)
{
    String payload = message.data();
    Serial.println("Received: " + payload);
    DynamicJsonDocument doc(1024);
    DeserializationError error = deserializeJson(doc, payload.c_str());

    if (error)
    {
        Serial.println("Failed to parse JSON");
        Serial.println(error.c_str());
        return;
    }

    const char *action = doc["action"];
    Serial.println(action);
    if (strcmp(action, "write") == 0)
    {
        upload = doc["upload"] | upload;
    }
    else if (strcmp(action, "read") == 0)
    {
        handleRead();
    }
}

// Function to initialize Wi-Fi
void setupWiFi()
{
    Serial.print("Connecting to Wi-Fi...");
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED)
    {
        delay(500);
        Serial.print(".");
    }
    Serial.println("\nConnected to Wi-Fi. IP: " + WiFi.localIP().toString());
}

// Function to initialize OTA
void setupOTA()
{
    ArduinoOTA.setPassword("password");

    ArduinoOTA.onStart([]()
    {
        String type = ArduinoOTA.getCommand() == U_FLASH ? "sketch" : "filesystem";
        Serial.println("Start updating " + type);
    });

    ArduinoOTA.onEnd([]()
    { Serial.println("\nUpdate Complete"); });

    ArduinoOTA.onProgress([](unsigned int progress, unsigned int total)
    { Serial.printf("Progress: %u%%\r", (progress / (total / 100))); });

    ArduinoOTA.onError([](ota_error_t error)
    {
        Serial.printf("Error[%u]: ", error);
        if (error == OTA_AUTH_ERROR) Serial.println("Authentication Failed");
        else if (error == OTA_BEGIN_ERROR) Serial.println("Begin Failed");
        else if (error == OTA_CONNECT_ERROR) Serial.println("Connect Failed");
        else if (error == OTA_RECEIVE_ERROR) Serial.println("Receive Failed");
        else if (error == OTA_END_ERROR) Serial.println("End Failed");
    });

    ArduinoOTA.begin();
    Serial.println("OTA Ready");
}

void setup()
{
    setupWiFi(); // Initialize Wi-Fi
    setupOTA();  // Initialize OTA

    webSocket.listen(80); // Start WebSocket server
    Serial.println("WebSocket server started. Connect to ws://" + WiFi.localIP().toString() + ":80");
}

void loop(void *parameter)
{
    for (;;)
    {
        if (!client.available() && !upload)
        {
            client = webSocket.accept();
            Serial.println("Client connected");
        }
        else if (!upload)
        {
            client.poll();
            auto message = client.readBlocking();
            handleWrite(message);
            Serial.println("Message received");
        }
        else if (upload)
        {
            if (client.available())
            {
                client.close();
                Serial.println("closed");
            }
            ArduinoOTA.handle(); // Handle OTA requests
        }

        vTaskDelay(100 / portTICK_PERIOD_MS); // Non-blocking delay
    }
}
```

