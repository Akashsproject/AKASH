#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <DHT.h>
#include <UniversalTelegramBot.h>

#define BLYNK_TEMPLATE_ID "TMPL3XX5AeqAu"
#define BLYNK_TEMPLATE_NAME "Temperature and Humidity"
#define BLYNK_AUTH_TOKEN "MfPc-bwUafZlhxmIYw0YplRFmaU9KwSV"
#define TELEGRAM_BOT_TOKEN "7182373445:AAFbTiOT73Frj9psuN24_ns0JNc2oVM3w1s"
#define TELEGRAM_CHAT_ID 1595763395 // Telegram Chat ID as an integer

#define BLYNK_PRINT Serial

char auth[] = BLYNK_AUTH_TOKEN;
char ssid[] = "Realme";
char pass[] = "spider11";
char telegramBotToken[] = TELEGRAM_BOT_TOKEN;
int telegramChatId = TELEGRAM_CHAT_ID;

BlynkTimer timer;

#define DHTPIN 4
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

WiFiClientSecure client;
UniversalTelegramBot bot(telegramBotToken, client);

void sendSensor() {
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  // Send temperature and humidity to Blynk app
  Blynk.virtualWrite(V0, t);
  Blynk.virtualWrite(V1, h);
  Serial.print("Temperature : ");
  Serial.print(t);
  Serial.print("    Humidity : ");
  Serial.println(h);
}

void handleMessages(int numNewMessages) {
  for (int i = 0; i < numNewMessages; i++) {
    String chatId = String(bot.messages[i].chat_id);
    String text = bot.messages[i].text;

    if (text == "/Temperature") {
      float temperature = dht.readTemperature();
      String response = "Current temperature: " + String(temperature) + " °C";
      bot.sendMessage(chatId, response);
    } else if (text == "/Humidity") {
      float humidity = dht.readHumidity();
      String response = "Current humidity: " + String(humidity) + " %";
      bot.sendMessage(chatId, response);
    }
  }
}

void setup() {
  Serial.begin(115200);

  Blynk.begin(auth, ssid, pass);
  dht.begin();
  timer.setInterval(100L, sendSensor);

  bot.setTelegramToken(telegramBotToken);
  timer.setInterval(1000L, handleMessages); // Check for new messages every second
}

void loop() {
  Blynk.run();
  timer.run();
}
