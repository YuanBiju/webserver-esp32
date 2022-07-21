# *** Still in development ***
# Relay control using ESP32
Lets get on with this project in which we control an led using a relay with commands given from a webpage.</br>
We'll go through the entire project step by step from creating a webpage to interfacing the esp32 with the relay.</br>

### Step1: Connecting esp32 to your wifi network
The following code connects the esp32 to you local wifi network which assigns it an ip address which the network will use to identify the esp32.
Read the comments in the code to understand what each piece of code do.

```c++
//include Wifi library
#include <WiFi.h>

// Replace with your network credentials
const char* ssid = "REPLACE_WITH_YOUR_SSID";
const char* password = "REPLACE_WITH_YOUR_PASSWORD";

// Set web server port number to 80
WiFiServer server(80);

void setup() {
  Serial.begin(115200);
  
  // Connect to Wi-Fi network with SSID and password
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  // start Http server
  startHttpServer();
  
  // Print local IP address and start web server
  Serial.println("");
  Serial.println("WiFi connected.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  server.begin();
}

void loop() {
  // put your main code here, to run repeatedly:
  delay(10000);
}
```

Why use port 80? Because we are creating a http server to communicate with the webpage and port 80 in a network is the port designated to handle HTTP requests.

### Step 2: Create http server
Open another tab in the same project in Arduino IDE and name it app_http_server.h
```c++
// include http server library
#include "esp_http_server.h"

// include the index html. this will be discussed later
#include "index_html.h"

// include library for logging
#include "esp_log.h"
static const char *TAG = "camera_httpd";

httpd_handle_t relay_httpd = NULL;

// handler to toggle led
static esp_err_t relay_handler(httpd_req_t *req)
{
    httpd_resp_set_type(req, "text/html");
    httpd_resp_set_hdr(req, "Content-Encoding", "gzip");
    sensor_t *s = esp_camera_sensor_get();
         
    return httpd_resp_send(req, (const char *)index_ov2640_html_gz, index_ov2640_html_gz_len);
}

// handler to serve index html page
static esp_err_t index_handler(httpd_req_t *req)
{
    httpd_resp_set_type(req, "text/html");
    httpd_resp_set_hdr(req, "Content-Encoding", "gzip");
    sensor_t *s = esp_camera_sensor_get();
         
    return httpd_resp_send(req, (const char *)index_ov2640_html_gz, index_ov2640_html_gz_len);
}

// function to set up http server
void startHttpServer()
{
    // use the default configuration for http server
    httpd_config_t config = HTTPD_DEFAULT_CONFIG();
    
    // create http uri struct for led toggling
    httpd_uri_t index_uri = {
        .uri = "/",
        .method = HTTP_GET,
        .handler = index_handler,
        .user_ctx = NULL};
        
    // create http uri struct for led toggling
    httpd_uri_t led_toggle_uri = {
        .uri = "/relay",
        .method = HTTP_GET,
        .handler = relay_handler,
        .user_ctx = NULL};
    
    ESP_LOGI(TAG, "Starting web server on port: '%d'", config.server_port);
    if (httpd_start(&relay_httpd, &config) == ESP_OK)
    {
        // register uri's with relay_httpd handler
        httpd_register_uri_handler(relay_httpd, &index_uri);
        httpd_register_uri_handler(relay_httpd, &led_toggle_uri);
    }
}
```

### Step 3: Create index html file
index.md file in this repository contains the index html gzip code to be used here.
  
```c++  
// File: index_ov2640.html.gz, Size: 6787
#define index_ov2640_html_gz_len 6787
const uint8_t index_ov2640_html_gz[] = {
 // copy and paste contents of index.md here
};
```

Compile and uplaod the code to esp32 board, open the serial terminal in Arduino and reboot the esp32.
The IP address of the esp32 board will appear on the terminal. Copy the address and search it in browser to get the index page from where you can control the led.
