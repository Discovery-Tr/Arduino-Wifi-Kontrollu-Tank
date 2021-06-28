#Arduino (NodeMCU) WİFİ KONTROLLÜ ARABA

https://kirmiziyuz.com/arduino/nodemcu-wifi-kontrollu-araba.html
NodeMCU ve L298N kullanarak wifi kontrollü araç yapımı.

#Malzemeler :

NodemCU
L298N motor sürücüsü
Robot kiti veya oyuncak araba
jumper kablolar
nodemcu ve l298n için 2 adet güç kaynağı (pil)

#Devre Şeması

![image alt text](https://www.kirmiziyuz.com/wp-content/uploads/2021/03/sema.png)

Web Sayfası Görünümü
![image alt text](https://www.kirmiziyuz.com/wp-content/uploads/2021/03/web.png)

#NodeMCU Programı

```
#include <ESP8266WiFi.h>
 
/* define port */
WiFiClient client;
WiFiServer server(80);
 
/* WIFI settings */
const char* ssid = "SSID";
const char* password = "Wifi Password";
 
/* data received from application */
String data ="";
 
/* define L298N or L293D motor control pins */
int leftMotorForward = 2; /* GPIO2(D4) -> IN3 */
int rightMotorForward = 15; /* GPIO15(D8) -> IN1 */
int leftMotorBackward = 0; /* GPIO0(D3) -> IN4 */
int rightMotorBackward = 13; /* GPIO13(D7) -> IN2 */
 
void setup() {
Serial.begin(115200);
delay(10);
 
/* initialize motor control pins as output */
pinMode(leftMotorForward, OUTPUT);
pinMode(rightMotorForward, OUTPUT);
pinMode(leftMotorBackward, OUTPUT);
pinMode(rightMotorBackward, OUTPUT);
 
/* Connect to WiFi network */
Serial.println();
Serial.println();
Serial.print("Connecting to ");
Serial.println(ssid);
 
WiFi.begin(ssid, password);
 
while (WiFi.status() != WL_CONNECTED) {
delay(500);
Serial.print(".");
}
Serial.println("");
Serial.println("WiFi connected");
 
/* start server communication */
server.begin();
Serial.println("Server started");
 
/* print the IP address */
Serial.print("Use this URL to connect: ");
Serial.print("http://");
Serial.print(WiFi.localIP());
Serial.println("/");
 
}
 
void loop() {
 
/* If the server available, run the "checkClient" function */
client = server.available();
if (!client) return;
data = checkClient ();
 
/************************ Run function according to incoming data from application *************************/
 
/* If the incoming data is "forward", run the "MotorForward" function */
if (data == "forward") MotorForward();
/* If the incoming data is "backward", run the "MotorBackward" function */
else if (data == "backward") MotorBackward();
/* If the incoming data is "left", run the "TurnLeft" function */
else if (data == "left") TurnLeft();
/* If the incoming data is "right", run the "TurnRight" function */
else if (data == "right") TurnRight();
/* If the incoming data is "stop", run the "MotorStop" function */
else if (data == "stop") MotorStop();
 
/* Web Browser */
client.println("<!DOCTYPE html>");
client.println("<html lang='en'>");
client.println("<head>");
client.println("<!-- Required meta tags -->");
client.println("<meta charset='utf-8'>");
client.println("<meta name='viewport' content='width=device-width, initial-scale=1, shrink-to-fit=no'>");
client.println("<!-- Bootstrap CSS -->");
client.println("<link rel='stylesheet' href='https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/css/bootstrap.min.css' integrity='sha384-rwoIResjU2yc3z8GV/NPeZWAv56rSmLldC3R/AZzGRnGxQQKnKkoFVhFQhNUwEyJ' crossorigin='anonymous'>");
client.println("</head>");
client.println("<body>");
client.println("<div class='container mt-2'>");
client.println("<div class='row'>");
client.println("<div class='col-sm-12'>");
client.println("<div class='card'>");
client.println("<div class='card-header'>");
client.println("<h4>NodeMCU Wifi Tank Controller</h4>");
client.println("</div>");
client.println("<div class='card-body'>");
client.println("<div class='table-responsive'>");
client.println("<thead>");
client.println("</thead>");
client.println("<tbody>");
client.println("<table width='40%' height='40%' border='0' cellspacing='0' cellpadding='0'align='center'>");
client.println("<tr>");
client.println("<td></td>");
client.println("<td class='align-center'><a href=\"/forward\"\"><div class='d-flex align-items-center justify-content-center  bg-light rounded' style='font-size: 7em'><svg class='bi bi-caret-up-fill' width='1em' height='1em' viewBox='0 0 16 16' fill='currentColor' xmlns='http://www.w3.org/2000/svg'><path d='M7.247 4.86l-4.796 5.481c-.566.647-.106 1.659.753 1.659h9.592a1 1 0 00.753-1.659l-4.796-5.48a1 1 0 00-1.506 0z'></path></svg></div></a></td>");
client.println("<td></td>");
client.println("</tr>");
client.println("<tr>");
client.println("<td class='align-center'><a href=\"/left\"\"><div class='d-flex align-items-center justify-content-center  bg-light rounded' style='font-size: 7em'><svg class='bi bi-caret-left-fill' width='1em' height='1em' viewBox='0 0 16 16' fill='currentColor' xmlns='http://www.w3.org/2000/svg'><path d='M3.86 8.753l5.482 4.796c.646.566 1.658.106 1.658-.753V3.204a1 1 0 00-1.659-.753l-5.48 4.796a1 1 0 000 1.506z'></path></svg></div></a></td>");
client.println("<td class='align-center'><a href=\"/stop\"\"><div class='d-flex align-items-center justify-content-center  bg-light rounded' style='font-size: 4em'><svg class='bi bi-x-octagon-fill' width='1em' height='1em' viewBox='0 0 16 16' fill='currentColor' xmlns='http://www.w3.org/2000/svg'><path fill-rule='evenodd' d='M11.46.146A.5.5 0 0011.107 0H4.893a.5.5 0 00-.353.146L.146 4.54A.5.5 0 000 4.893v6.214a.5.5 0 00.146.353l4.394 4.394a.5.5 0 00.353.146h6.214a.5.5 0 00.353-.146l4.394-4.394a.5.5 0 00.146-.353V4.893a.5.5 0 00-.146-.353L11.46.146zm.394 4.708a.5.5 0 00-.708-.708L8 7.293 4.854 4.146a.5.5 0 10-.708.708L7.293 8l-3.147 3.146a.5.5 0 00.708.708L8 8.707l3.146 3.147a.5.5 0 00.708-.708L8.707 8l3.147-3.146z' clip-rule='evenodd'></path></svg></div></a></td>");
client.println("<td class='align-center'><a href=\"/right\"\"><div class='d-flex align-items-center justify-content-center bg-light rounded' style='font-size: 7em'><svg class='bi bi-caret-right-fill' width='1em' height='1em' viewBox='0 0 16 16' fill='currentColor' xmlns='http://www.w3.org/2000/svg'><path d='M12.14 8.753l-5.482 4.796c-.646.566-1.658.106-1.658-.753V3.204a1 1 0 011.659-.753l5.48 4.796a1 1 0 010 1.506z'></path></svg></div></a></td>");
client.println("</tr>");
client.println("<tr>");
client.println("<td> </td>");
client.println("<td class='align-center'><a href=\"/backward\"\"><div class='d-flex align-items-center justify-content-center bg-light rounded' style='font-size: 7em'><svg class='bi bi-caret-down-fill' width='1em' height='1em' viewBox='0 0 16 16' fill='currentColor' xmlns='http://www.w3.org/2000/svg'><path d='M7.247 11.14L2.451 5.658C1.885 5.013 2.345 4 3.204 4h9.592a1 1 0 01.753 1.659l-4.796 5.48a1 1 0 01-1.506 0z'></path></svg></div></a></td>");
client.println("<td></td>");
client.println("</tr>");
client.println("</table>");
client.println("</tbody>");
client.println("</div>");
client.println("</div>");
client.println("<div class='card-footer'>");
client.println("<div class='form-group float-left'>Adem KIRMIZIYÜZ 2020</div>");
client.println("</div>");
client.println("</div>");
client.println("</div>");
client.println("</div>");
client.println("</body>");
client.println("</html>");
}
 

 
/********************************************* FORWARD *****************************************************/
void MotorForward(void)
{
digitalWrite(leftMotorForward,HIGH);
digitalWrite(rightMotorForward,HIGH);
digitalWrite(leftMotorBackward,LOW);
digitalWrite(rightMotorBackward,LOW);
}
 
/********************************************* BACKWARD *****************************************************/
void MotorBackward(void)
{
digitalWrite(leftMotorBackward,HIGH);
digitalWrite(rightMotorBackward,HIGH);
digitalWrite(leftMotorForward,LOW);
digitalWrite(rightMotorForward,LOW);
}
 
/********************************************* TURN LEFT *****************************************************/
void TurnLeft(void)
{
digitalWrite(leftMotorForward,LOW);
digitalWrite(rightMotorForward,HIGH);
digitalWrite(rightMotorBackward,LOW);
digitalWrite(leftMotorBackward,HIGH);
}
 
/********************************************* TURN RIGHT *****************************************************/
void TurnRight(void)
{
digitalWrite(leftMotorForward,HIGH);
digitalWrite(rightMotorForward,LOW);
digitalWrite(rightMotorBackward,HIGH);
digitalWrite(leftMotorBackward,LOW);
}
 
/********************************************* STOP *****************************************************/
void MotorStop(void)
{
digitalWrite(leftMotorForward,LOW);
digitalWrite(leftMotorBackward,LOW);
digitalWrite(rightMotorForward,LOW);
digitalWrite(rightMotorBackward,LOW);
}
 
/********************************** RECEIVE DATA FROM the WEB ******************************************/
String checkClient (void)
{
while(!client.available()) delay(1);
String request = client.readStringUntil('\r');
request.remove(0, 5);
request.remove(request.length()-9,9);
return request;
}
```
