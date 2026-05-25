# IoT-Project


Tecnologias utilizadas 

 node-red 
 mqtt 
 Wokwi ESP32 
InfluxDB 
Grafana 
Telegram para notificações 


fluxo: ESP32 (consulta api do tempo) --> node-red ( dentro de uma EC2 na aws) --> execução do fluxo com a logica do projeto --> Salva informações no InfluxDB --> Grafana --> Dashboard
fluxo: ESP32 (consulta api do tempo) --> node-red ( dentro de uma EC2 na aws) --> execução do fluxo com a logica do projeto --> mandar de volta a informação da abertura/fechamneto da bomba hidrica ao mqtt --> ESP32 (executa)
fluxo: ESP32 (consulta api do tempo) --> node-red ( dentro de uma EC2 na aws) --> execução do fluxo com a logica do projeto --> telegram (manda as informações do status atual par ao cliente via bot)
