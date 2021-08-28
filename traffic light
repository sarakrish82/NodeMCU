include "TrafficLight.h"
include <ESP8266WiFi.h>
include <ESP8266WebServer.h>

char ssid[] = "<your-wifi-ssid>";
char pass[] = "<your-wifi-password>";

// create a server with port 80 (default web port)
ESP8266WebServer  server(80);
// this will hold the current traffic light max
int ct = 0;
// set a maximum of 3 because of memory & pin limitations
TrafficLight *trafficLights[3];

void connectWIFI() {
  // for decoration purposes
  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  // try to connect to the wifi
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    // retry attempt every 500ms
    // we won't start our code if wifi hasn't connected
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected!");
  // shows the IP address that our device uses
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}

// returnInvalid will return a response with 400 Invalid Request
void returnInvalid() {
  server.send(400, "text/plain", "Invalid Request");
}

// returnOK will return a response with 200 OK
void returnOK() {
  server.send(200, "text/plain", "OK");
}

// handleTraffic will handle requests to change a specific traffic light object's state
void handleTraffic() {
  if(!server.hasArg("state") || !server.hasArg("id")) {
     returnInvalid();
  }
  int id = server.arg("id").toInt();
  if(trafficLights[id]==NULL) {
    returnInvalid();
  }
  switch(server.arg("state").toInt()) {
    case RED:
      trafficLights[id]->Stop();
      break;
    case GREEN:
      trafficLights[id]->Go();
      break;
    case YELLOW:
      trafficLights[id]->Careful();
      break;
    default:
      returnInvalid();
  }
  returnOK();
}

// handleRegister handles requests to register a new traffic light object provided the LED pins
void handleRegister() {
  if(!server.hasArg("redPin") || !server.hasArg("yellowPin") || !server.hasArg("greenPin")) {
    returnInvalid();
  }
  if(trafficLights[ct]!=NULL) {
    free(trafficLights[ct]);
  }
  TrafficLight* tl = new TrafficLight(server.arg("redPin").toInt(), server.arg("yellowPin").toInt(), server.arg("greenPin").toInt());

  if(server.hasArg("id")){
    int id = server.arg("id").toInt();
    if(id<3 && id>=0) {
       ct = id;
     } else {
       returnInvalid();
     }
  }
  trafficLights[ct] = tl;
  server.send(200, "text/plain", String(ct));
  ct = (ct + 1)%3;
}

// startServer will register the handlers and start the server
void startServer() {
  server.on("/traffic", handleTraffic);
  server.on("/register", handleRegister);
  server.begin(); //Start the server
  Serial.println("Server listening");
}

// we will call this function first
void setup() {
  Serial.begin(115200);
  delay(10);
  // this section is optional, but I find it being more consistent
  WiFi.mode(WIFI_OFF);
  delay(1000);
  WiFi.mode(WIFI_STA);
  while(!Serial);
  connectWIFI();
  startServer();
}

void loop() {
  //Handling of incoming client requests
  server.handleClient(); 
}
