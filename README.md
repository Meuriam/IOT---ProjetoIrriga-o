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

Arquivo: 

# Comunicação MQTT

O Sub tópico jardim/bomba: neste tópico, a placa ESP8266 busca e retorna o valor atual da umidade do solo, sendo mostrado na dashboard instalado no celular(MQTTDashboard).
