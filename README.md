# IoT Project - Sistema Inteligente de Irrigação com ESP32, Node-RED, InfluxDB, Grafana e Telegram

## Sumário

- [1. Visão geral](#1-visão-geral)
- [2. Objetivo do projeto](#2-objetivo-do-projeto)
- [3. Contexto da solução](#3-contexto-da-solução)
- [4. Tecnologias utilizadas](#4-tecnologias-utilizadas)
- [5. Arquitetura do projeto](#5-arquitetura-do-projeto)
- [6. Fluxo geral de funcionamento](#6-fluxo-geral-de-funcionamento)
- [7. Lógica de decisão da irrigação](#7-lógica-de-decisão-da-irrigação)
- [8. Estrutura da solução](#8-estrutura-da-solução)
- [9. Simulações no Wokwi](#9-simulações-no-wokwi)
- [10. Como recriar o projeto](#10-como-recriar-o-projeto)
  - [10.1 Criar a instância EC2](#101-criar-a-instância-ec2)
  - [10.2 Conectar na instância](#102-conectar-na-instância)
  - [10.3 Atualizar o sistema](#103-atualizar-o-sistema)
  - [10.4 Instalar Node.js](#104-instalar-nodejs)
  - [10.5 Instalar e executar o Node-RED](#105-instalar-e-executar-o-node-red)
  - [10.6 Configurar inicialização automática com PM2](#106-configurar-inicialização-automática-com-pm2)
  - [10.7 Liberar acesso ao Node-RED](#107-liberar-acesso-ao-node-red)
  - [10.8 Configurar o broker MQTT](#108-configurar-o-broker-mqtt)
  - [10.9 Configurar o InfluxDB](#109-configurar-o-influxdb)
  - [10.10 Configurar o Grafana](#1010-configurar-o-grafana)
  - [10.11 Configurar o Telegram](#1011-configurar-o-telegram)
  - [10.12 Importar e ajustar o fluxo no Node-RED](#1012-importar-e-ajustar-o-fluxo-no-node-red)
- [11. Fluxo Node-RED](#11-fluxo-node-red)
- [12. Dashboard no Grafana](#12-dashboard-no-grafana)
- [13. Integração com InfluxDB](#13-integração-com-influxdb)
- [14. Estrutura das mensagens processadas](#14-estrutura-das-mensagens-processadas)
- [15. Exemplo de notificação no Telegram](#15-exemplo-de-notificação-no-telegram)
- [16. Problemas comuns e soluções](#16-problemas-comuns-e-soluções)
- [17. Resultados esperados](#17-resultados-esperados)
- [18. Melhorias futuras](#18-melhorias-futuras)
- [19. Conclusão](#19-conclusão)

---

## 1. Visão geral

Este projeto apresenta uma solução de **IoT aplicada à irrigação inteligente**, integrando dispositivos, serviços em nuvem, automação de fluxo, banco de dados de séries temporais, dashboards e notificações em tempo real.

A proposta é usar um **ESP32** para obter informações climáticas e disparar o processamento no **Node-RED**, que executa a lógica principal do sistema. A partir disso, o projeto:

- analisa variáveis ambientais;
- decide se a irrigação deve ser ligada ou desligada;
- envia o comando para o sistema de acionamento;
- registra os dados históricos no **InfluxDB**;
- exibe os resultados no **Grafana**;
- envia notificações ao usuário via **Telegram**.

O projeto foi construído com foco em **automação**, **monitoramento remoto** e **tomada de decisão baseada em dados**.

---

## 2. Objetivo do projeto

O objetivo do projeto é desenvolver um sistema capaz de **automatizar decisões de irrigação** a partir de dados de clima e solo, simulando um cenário real de agricultura inteligente.

Mais especificamente, a solução busca:

- coletar dados de clima e umidade do solo;
- processar as informações em tempo real;
- calcular a necessidade hídrica da cultura;
- decidir quando irrigar;
- armazenar o histórico de medições;
- apresentar dashboards de acompanhamento;
- notificar o usuário sobre o estado do sistema.

---

## 3. Contexto da solução

Em cenários agrícolas, a irrigação inadequada pode gerar desperdício de água, aumento de custos e prejuízo ao desenvolvimento da plantação.  
Por isso, sistemas inteligentes de irrigação se tornam importantes para apoiar decisões mais eficientes.

Neste projeto, o sistema considera fatores como:

- **cidade monitorada**;
- **tipo de cultura associada à cidade**;
- **umidade do solo**;
- **temperatura/sensação térmica**;
- **previsão de chuva para as próximas 24 horas**.

Com base nesses dados, o Node-RED calcula a **necessidade hídrica ideal** e define se a irrigação deve ser ativada ou não.

---

## 4. Tecnologias utilizadas

As tecnologias e serviços utilizados foram:

- **ESP32**
- **Wokwi**
- **MQTT**
- **Node-RED**
- **AWS EC2**
- **OpenWeatherMap API**
- **AgroMonitoring API**
- **InfluxDB**
- **Grafana**
- **Telegram Bot**

---

## 5. Arquitetura do projeto

A arquitetura é composta por cinco camadas principais:

### 1. Camada de coleta
O **ESP32** simulado no Wokwi envia a cidade para iniciar o processamento.

### 2. Camada de processamento
O **Node-RED**, hospedado em uma **instância EC2 da AWS**, concentra toda a lógica do projeto.

### 3. Camada de comunicação
A troca de mensagens é feita via **MQTT**, utilizando o broker público **HiveMQ**.

### 4. Camada de persistência
Os dados processados são gravados no **InfluxDB**, permitindo armazenamento histórico.

### 5. Camada de visualização e alerta
O **Grafana** apresenta dashboards para monitoramento, enquanto o **Telegram** envia notificações automáticas.

---

## 6. Fluxo geral de funcionamento

O fluxo principal do sistema segue esta sequência:

**ESP32 -> MQTT -> Node-RED -> APIs externas -> lógica de irrigação -> MQTT / InfluxDB / Telegram / Grafana**

### Etapas do fluxo

1. O **ESP32** publica uma mensagem MQTT contendo a cidade.
2. O **Node-RED** recebe essa mensagem pelo tópico:
   - `MACK10438932/Temperatura`
3. O fluxo consulta a **API AgroMonitoring** para obter dados de umidade do solo.
4. Em seguida, consulta a **OpenWeatherMap Forecast API** para previsão de chuva.
5. Depois, consulta a **OpenWeatherMap Current Weather API** para obter:
   - sensação térmica;
   - clima atual;
   - vento;
   - cidade e país.
6. O Node-RED executa a lógica de decisão.
7. O sistema:
   - publica o comando de irrigação no tópico MQTT `MACK/ComandoIrrigacao`;
   - salva os dados no InfluxDB;
   - envia uma mensagem no Telegram.

---

## 7. Lógica de decisão da irrigação

A lógica implementada no Node-RED considera:

- **umidade do solo**
- **previsão de chuva nas próximas 24 horas**
- **temperatura/sensação térmica**
- **cultura agrícola relacionada à cidade**

### Associação cidade -> cultura

O fluxo possui um mapeamento inicial:

- **São Paulo** -> milho
- **Campinas** -> café
- **São José dos Campos** -> algodão

Caso a cidade não esteja no mapa, o sistema assume a cultura **milho** como padrão.

### Necessidade hídrica base por cultura

Exemplos definidos no fluxo:

- milho -> 6 mm
- soja -> 5 mm
- café -> 4 mm
- cana -> 7 mm
- tomate -> 8 mm
- alface -> 5 mm
- arroz -> 9 mm
- feijão -> 5 mm
- trigo -> 4 mm
- algodão -> 6 mm

### Ajuste pela temperatura

A necessidade hídrica é ajustada conforme a temperatura:

- temperatura >= 35°C -> +3 mm
- temperatura >= 30°C -> +2 mm
- temperatura >= 25°C -> +1 mm

### Regras de decisão

A irrigação é ativada ou desativada com base nestas condições:

1. **Se a umidade do solo for menor ou igual a 25%**
   - irrigação = **ligada**
   - motivo: solo extremamente seco

2. **Se a previsão de chuva em 24h for maior ou igual à necessidade hídrica**
   - irrigação = **desligada**
   - motivo: previsão de chuva suficiente

3. **Se a umidade do solo for menor ou igual a 45% e a previsão de chuva for insuficiente**
   - irrigação = **ligada**
   - motivo: baixa umidade e ausência de chuva

4. **Caso contrário**
   - irrigação = **desligada**
   - motivo: solo já úmido

---

## 8. Estrutura da solução

A solução foi dividida nos seguintes componentes:

### ESP32
Responsável por simular o dispositivo que inicia o processo e participa da automação.

### MQTT
Responsável pela comunicação entre os elementos do sistema.

### Node-RED
Responsável pela orquestração de toda a lógica:
- consumo de mensagens MQTT;
- chamadas HTTP para APIs;
- decisão de irrigação;
- envio de notificação;
- persistência no banco.

### InfluxDB
Responsável por armazenar:
- temperatura;
- umidade do solo;
- previsão de chuva;
- vento;
- estado da irrigação;
- necessidade hídrica;
- data e hora da leitura;
- cidade, cultura e clima.

### Grafana
Responsável por transformar os dados históricos em painéis visuais.

### Telegram
Responsável por enviar alertas e mensagens informativas ao usuário final.

---

## 9. Simulações no Wokwi

Os projetos utilizados no Wokwi são:

- **ESP32 que consulta dados do tempo**  
  `https://wokwi.com/projects/463754552975480833`

- **ESP32 que executa a abertura e fechamento da bomba hidráulica**  
  `https://wokwi.com/projects/464740954234441729`

Essas simulações permitem reproduzir o comportamento do sistema sem necessidade de hardware físico real.

---

## 10. Como recriar o projeto

A seguir está um passo a passo detalhado para reproduzir a solução.

---

### 10.1 Criar a instância EC2

Criar uma instância com as configurações:

- **Ubuntu 24.04 LTS (HVM)**
- **t2.medium**
- criar novo par de chaves `.pem`
- liberar acesso:
  - SSH
  - HTTP
  - HTTPS

Se desejar usar PuTTY, converter o arquivo `.pem` para `.ppk` com o **PuTTYgen**.

---

### 10.2 Conectar na instância

No AWS CloudShell, subir a chave `.pem` e executar:

```bash
chmod 400 <arquivo.pem>
ssh -i "<arquivo.pem>" <nome-FQDN-ou-IP-da-instancia>
```

---

### 10.3 Atualizar o sistema

```bash
sudo apt update
sudo apt upgrade -y
```

---

### 10.4 Instalar Node.js

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash
\. "$HOME/.nvm/nvm.sh"
nvm install 25
sudo apt install libatomic1
```

---

### 10.5 Instalar e executar o Node-RED

```bash
npm install -g node-red
npm install -g npm@11.12.1
node-red
```

Acesso padrão:

```text
http://IP-DA-INSTANCIA:1880
```

---

### 10.6 Configurar inicialização automática com PM2

```bash
npm install -g pm2
pm2 start node-red
pm2 startup
sudo env PATH=$PATH:/home/ubuntu/.nvm/versions/node/v25.8.2/bin /home/ubuntu/.nvm/versions/node/v25.8.2/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
pm2 save
```

---

### 10.7 Liberar acesso ao Node-RED

Para redirecionar a porta 80 para a 1880:

```bash
sudo iptables -t nat -I PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 1880
```

Alternativamente, liberar diretamente a porta **1880** no grupo de segurança da instância EC2.

---

### 10.8 Configurar o broker MQTT

O projeto utiliza o broker público:

```text
broker.hivemq.com
```

Configuração usada no Node-RED:

- broker: `broker.hivemq.com`
- porta: `1883`
- protocolo: MQTT v3.1.1 / v4
- tópico de entrada:
  - `MACK10438932/Temperatura`
- tópico de saída:
  - `MACK/ComandoIrrigacao`

---

### 10.9 Configurar o InfluxDB

O Node-RED grava os dados em um bucket do InfluxDB 2.0.

Configuração observada no fluxo:

- versão: **InfluxDB 2.0**
- URL: `https://us-east-1-1.aws.cloud2.influxdata.com`
- bucket: `IoT`
- measurement: `Temperatura`

Documentação da API:

```text
https://docs.influxdata.com/influxdb3/cloud-serverless/api/#operation/PostWrite
```

### Campos armazenados
- `temperatura`
- `umidade_solo`
- `previsao_chuva`
- `vento`
- `irrigacao`
- `necessidade_hidrica`
- `data_leitura`
- `hora_leitura`

### Tags armazenadas
- `cidade`
- `cultura`
- `clima`

---

### 10.10 Configurar o Grafana

No Grafana:

1. adicionar o InfluxDB como fonte de dados;
2. configurar o acesso ao bucket do projeto;
3. criar dashboards para monitoramento.

### Sugestão de dashboards

#### Painel 1 - Temperatura
Exibir a temperatura/sensação térmica ao longo do tempo.

#### Painel 2 - Umidade do solo
Exibir a variação da umidade do solo.

#### Painel 3 - Previsão de chuva
Exibir os milímetros previstos para as próximas 24 horas.

#### Painel 4 - Estado da irrigação
Exibir se a irrigação está ligada ou desligada.

#### Painel 5 - Necessidade hídrica
Exibir a necessidade hídrica calculada.

#### Painel 6 - Histórico por cidade/cultura
Permitir filtrar os dados por cidade, cultura e clima.

---

### 10.11 Configurar o Telegram

O fluxo usa um **Telegram Bot** para notificar o usuário.

Etapas:

1. criar um bot com o **BotFather**;
2. obter o token do bot;
3. descobrir o `chatId`;
4. configurar o nó `telegram sender` no Node-RED;
5. ajustar a função que monta a mensagem.

O conteúdo da mensagem enviada inclui:

- cidade;
- status da irrigação;
- umidade do solo;
- previsão de chuva;
- temperatura;
- cultura;
- necessidade hídrica;
- motivo da decisão.

---

### 10.12 Importar e ajustar o fluxo no Node-RED

Depois de instalar o Node-RED:

1. abrir o editor;
2. importar o JSON do fluxo;
3. revisar as credenciais e integrações;
4. ajustar chaves e tokens;
5. fazer o deploy.

### Pontos que precisam ser revisados no fluxo
Ao importar o projeto, verificar:

- chave da **OpenWeatherMap API**
- chave da **AgroMonitoring API**
- credenciais do **InfluxDB**
- configuração do **Telegram Bot**
- `chatId` do Telegram
- tópicos MQTT
- cidades e culturas associadas

---

## 11. Fluxo Node-RED

Abaixo está a imagem do fluxo principal no Node-RED:

<img width="1566" height="239" alt="Fluxo do Node-RED" src="https://github.com/user-attachments/assets/e0774d41-f0fd-49f5-9ed8-41e9ff9d01c7" />

### O que esse fluxo faz

O fluxo executa as seguintes etapas:

1. recebe a cidade via MQTT;
2. salva a cidade no contexto do fluxo;
3. consulta a API AgroMonitoring para obter umidade do solo;
4. consulta a previsão do tempo na OpenWeatherMap;
5. consulta o clima atual;
6. calcula a necessidade hídrica;
7. decide se a irrigação deve ser ativada;
8. envia o comando via MQTT;
9. grava os dados no InfluxDB;
10. envia a notificação via Telegram.

---

## 12. Dashboard no Grafana

Abaixo está um exemplo de dashboard utilizado no projeto:

<img width="1302" height="745" alt="Dashboard do Grafana" src="https://github.com/user-attachments/assets/21fd47c0-ac90-40f7-80cd-7928ecbc6941" />

### Sugestão de organização do dashboard

Para reproduzir um dashboard semelhante, recomenda-se criar painéis com:

- temperatura atual;
- umidade do solo;
- previsão de chuva;
- velocidade do vento;
- estado da irrigação;
- necessidade hídrica;
- histórico temporal dos eventos.

---

## 13. Integração com InfluxDB

Abaixo está a referência visual da escrita/integracao com o InfluxDB:

<img width="1001" height="415" alt="Integração com InfluxDB" src="https://github.com/user-attachments/assets/d935a57b-ef2e-46e4-be3a-c467cef0527d" />

O InfluxDB é responsável por armazenar os dados processados pelo Node-RED, permitindo consultas posteriores e construção dos dashboards no Grafana.

---

## 14. Estrutura das mensagens processadas

O fluxo final monta uma mensagem contendo:

- `irrigacao`
- `rele`
- `dados.cidade`
- `dados.cultura`
- `dados.umidadeSolo`
- `dados.temperatura`
- `dados.previsaoChuva`
- `dados.necessidadeHidrica`
- `mensagem`
- `timestamp`

Exemplo conceitual:

```json
{
  "irrigacao": true,
  "rele": "ON",
  "dados": {
    "cidade": "Campinas",
    "cultura": "cafe",
    "umidadeSolo": 22.5,
    "temperatura": 31.4,
    "previsaoChuva": 1.2,
    "necessidadeHidrica": 6
  },
  "mensagem": "Baixa umidade e ausência de chuva.",
  "timestamp": "2026-05-25T12:00:00.000Z"
}
```

---

## 15. Exemplo de notificação no Telegram

O Telegram recebe uma mensagem parecida com esta:

```text
🌱 SISTEMA DE IRRIGAÇÃO

🏙️ Cidade: Campinas

🚿 Irrigação: ATIVADA

💧 Umidade do solo: 22.5%
🌧️ Previsão de chuva: 1.2mm
🌡️ Temperatura: 31.4°C
🌾 Cultura: cafe
💦 Necessidade hídrica: 6mm

📝 Motivo:
Baixa umidade e ausência de chuva.
```

---

## 16. Problemas comuns e soluções

### Node-RED não inicia após reinicialização
Configurar o PM2 corretamente.

### Não consigo acessar o Node-RED
Verificar:
- se a instância EC2 está ativa;
- se a porta 1880 está liberada;
- se o redirecionamento da porta 80 foi aplicado;
- se o processo `node-red` está rodando.

### Erro "Lost connection to server"
Isso pode ocorrer devido ao uso de WebSockets em redes instáveis.  
Teste em outra rede e revise firewall, portas e estabilidade da instância.

### APIs não retornam dados
Verificar:
- validade das chaves;
- limites de uso;
- cidade enviada;
- formato da URL;
- conectividade da instância com a internet.

### Telegram não envia mensagens
Verificar:
- token do bot;
- `chatId`;
- configuração do nó do Telegram no Node-RED.

### InfluxDB não grava dados
Verificar:
- URL;
- bucket;
- token;
- organização;
- conectividade externa.

---

## 17. Resultados esperados

Ao finalizar a configuração, o sistema deve ser capaz de:

- receber a cidade via MQTT;
- consultar dados ambientais;
- calcular a necessidade hídrica;
- decidir automaticamente pela irrigação;
- enviar comandos para o sistema atuador;
- registrar os dados em banco;
- gerar dashboards de acompanhamento;
- notificar o usuário em tempo real.

---

## 18. Conclusão

Este projeto demonstra como diferentes tecnologias podem ser integradas em uma solução completa de **IoT para irrigação inteligente**.

A combinação entre **ESP32**, **MQTT**, **Node-RED**, **APIs climáticas**, **InfluxDB**, **Grafana** e **Telegram** mostra uma arquitetura moderna, modular e escalável, capaz de automatizar decisões, registrar histórico e fornecer monitoramento em tempo real.

Além de ser uma prova de conceito funcional, o projeto também serve como base para futuras evoluções em cenários reais de agricultura inteligente e automação remota.
