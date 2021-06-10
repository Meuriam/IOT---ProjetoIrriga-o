# IOT ProjetoIrrigação Automática

Projeto da disciplina Objetos Inteligentes Conectados

Meuriam Assis e Thiago Siqueira Garbuio

O objetivo deste projeto, foi construir um sistema de irrigação automátia utilizando o módulo NodeMCU ESP8266 com o Sensor Higrômetro que mede a umidade do solo, acionando quando necessário uma Mini Bomba de Água através de um Relé que irá alimentar com uma mangueira o solo com água. Tudo foi controlado via protocolo MQTT, podendo acessar uma dashboard.

# Componentes

•	NodeMCU ESP8266
•	Sensor de Umidade
•	Protoboard 400 Pontos
•	Módulo Relé 1 Canal
•	Jumpers
•	USB
•	Mini Bomba de Água 5V
•	Mangueira Aquário
•	LEDs

# Código
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

//Definindo as variáveis e os pinos e entradas correspondentes de cada componente
#define DEBUG
#define pino_sinal_analogico A0

#define pino_led_vermelho 13 //5
#define pino_led_verde 12 //7
#define pino_bomba 14 //8 14

//Variáveis
int valor_analogico;
String strMSG = "0";
char mensagem[30];

//Informações da Rede WIFI para conexão com o CloudMQTT, que estabelece conectividade com o Broker que fará o monitoramento dos Solos
const char* ssid = "Bravo 007";             //SSID da rede WIFI
const char* password =  "mercado088";    //senha da rede wifi
//Informações da Instância do broker MQTT
const char* mqttServer = "driver.cloudmqtt.com";   //server
const char* mqttUser = "lmoxbnos";              //user
const char* mqttPassword = "xxXwgNY7ACec";      //password
const int mqttPort = 18936;                     //port
const char* mqttTopicSub = "jardim/bomba";      //tópico que será assinado no Broker

WiFiClient espClient;
PubSubClient client(espClient);

//Definindo as entradas correspondentes aos pinos e os Prints de Resposta para o monitor serial
void setup() {
  Serial.begin(9600);
  pinMode(pino_sinal_analogico, INPUT);
  pinMode(pino_led_vermelho, OUTPUT);
  pinMode(pino_led_verde, OUTPUT);
  pinMode(pino_bomba, OUTPUT);

  WiFi.begin(ssid, password);

  Serial.print("Entrando no Setup");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
#ifdef DEBUG
    Serial.println("Aguarde! Conectando ao WiFi...");
#endif
  }
#ifdef DEBUG
  Serial.println("Conectado na rede WiFi com sucesso!");
#endif

  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);

  while (!client.connected()) {
#ifdef DEBUG
    Serial.println("Conectando ao Broker MQTT...");
#endif

    if (client.connect("ESP8266Client", mqttUser, mqttPassword )) {
#ifdef DEBUG
      Serial.println("Conectado com sucesso");
#endif

    } else {
#ifdef DEBUG
      Serial.print("falha estado  ");
      Serial.print(client.state());
#endif
      delay(2000);
    }
  }

  //subscreve no tópico
  client.subscribe(mqttTopicSub);

}

void loop() {

  if (!client.connected()) {
    Serial.print("Passou no Connected");
    reconect();

  }
 
  Serial.print(valor_analogico);
  //Lendo o valor do pino A0 do sensor para detecção
  valor_analogico = analogRead(pino_sinal_analogico);

  //Mostrando o valor da porta analógica no serial monitor
  Serial.print("Porta analogica: ");
  Serial.print(valor_analogico);

  //Definindo o parâmetro para um Solo devidamente úmido, acendendo o LED VERDE
    if (valor_analogico >= 0 && valor_analogico < 500)
    {
      Serial.println("Status: Solo umido");
    
      digitalWrite (pino_led_vermelho, LOW);
      digitalWrite (pino_led_verde, HIGH);
      digitalWrite (pino_bomba, HIGH);
  
      //Enviando Mensagem ao Broker
      sprintf(mensagem, "0"); //Definindo o valor zero como parâmetro para o Broker mostrar uma imagem "vermelha"
      Serial.print("Mensagem enviada: ");
      Serial.println(mensagem);
      client.publish("jardim/bomba", mensagem);
      Serial.println("Mensagem enviada com sucesso...");
  
    }

 
  //Definindo o parâmetro para um Solo Seco, acendendo o LED VERMELHO
    if (valor_analogico >= 500 && valor_analogico <= 1024)
    {
      Serial.println(" Status: Solo seco");
      
      digitalWrite (pino_led_verde, LOW);
      digitalWrite (pino_led_vermelho, HIGH);
      digitalWrite (pino_bomba, LOW);
  
       //Enviando Mensagem ao Broker
      sprintf(mensagem, "1"); //Definindo o valor UM como parâmetro para o Broker mostrar uma imagem "verde"
      Serial.print("Mensagem enviada: ");
      Serial.println(mensagem); 
      client.publish("jardim/bomba", mensagem);
      Serial.println("Mensagem enviada com sucesso...");
  
    }
  delay(10000);

  client.loop();

}

void callback(char* topic, byte* payload, unsigned int length) {

  //Armazenando mensagem recebida em uma string
  payload[length] = '\0';
  strMSG = String((char*)payload);

#ifdef DEBUG
  Serial.print("Mensagem chegou do tópico: ");
  Serial.println(topic);
  Serial.print("Mensagem:");
  Serial.print(strMSG);
  Serial.println();
  Serial.println("-----------------------");
#endif

}

//Função pra reconectar ao servidor MQTT
void reconect() {
  //Enquanto estiver desconectado
  while (!client.connected()) {
#ifdef DEBUG
    Serial.print("Tentando conectar ao servidor MQTT");
#endif

    bool conectado = strlen(mqttUser) > 0 ?
                     client.connect("ESP8266Client", mqttUser, mqttPassword) :
                     client.connect("ESP8266Client");

    if (conectado) {
#ifdef DEBUG
      Serial.println("Conectado!");
#endif
      //Subscreve no tópico
      client.subscribe(mqttTopicSub, 1); //nivel de qualidade: QoS 1
    } else {
#ifdef DEBUG
      Serial.println("Falha durante a conexão.Code: ");
      Serial.println( String(client.state()).c_str());
      Serial.println("Tentando novamente em 10 s");
#endif
      //Aguarda 10 segundos
      delay(5000);
    }
  }
}
# Comunicação MQTT

O Sub tópico jardim/bomba: neste tópico, a placa ESP8266 busca e retorna o valor atual da umidade do solo, sendo mostrado na dashboard instalado no celular(MQTTDashboard).
