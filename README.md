
## RM's 👤
- `Caio Suzano Ferreira da Silva: RM 554763`
- `Matheus Rivera Montovaneli: RM 555499`
- `Lucas Vasquez Silva: RM 555159`
- `Guilherme Linard F. R. Gozzi: RM 555768`
- `André Nakamatsu Rocha: RM 555004`

# Projeto: Leitura de Sensores com ESP32 e LDR para Publicação via MQTT no Node-RED (via Wokwi)

Este projeto demonstra como utilizar o **ESP32** no **Wokwi** para ler os sensores  **DHT22** e **LDR** e enviar dados para um broker **MQTT**. Os dados podem ser consumidos e visualizados no **Node-RED** para automação e monitoramento em tempo real.

---

## 🛠️ Pré-requisitos

### O que é o Node-RED?
O **Node-RED** é uma ferramenta de desenvolvimento baseada em fluxos visuais, projetada para integrar hardware, APIs e serviços. Ele permite criar automações e aplicações IoT facilmente, arrastando e conectando blocos lógicos (nós) em uma interface gráfica.

- **Como Funciona**:
  - Conecta sensores, APIs e sistemas usando **blocos visuais**.
  - Recebe dados de dispositivos via **MQTT**.
  - Permite visualizar, armazenar e criar automações com dados recebidos.

---

### Instalação do Node-RED

#### **Windows**
1. **Instale o Node.js** (requisito do Node-RED):
   - Baixe e instale: [https://nodejs.org](https://nodejs.org).
2. Abra o **Prompt de Comando** e execute:
   ```bash
   npm install -g --unsafe-perm node-red
   ```
3. Execute o Node-RED com o comando:
   ```bash
   node-red
   ```
4. Acesse no navegador: [http://localhost:1880](http://localhost:1880).

#### **Linux/Ubuntu**
1. Atualize pacotes e instale o Node.js:
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install nodejs npm -y
   ```
2. Instale o Node-RED:
   ```bash
   sudo npm install -g --unsafe-perm node-red
   ```
3. Execute o Node-RED:
   ```bash
   node-red
   ```
4. Acesse: [http://localhost:1880](http://localhost:1880).

#### **MacOS**
1. Instale o Node.js via Homebrew:
   ```bash
   brew install node
   ```
2. Instale o Node-RED:
   ```bash
   npm install -g --unsafe-perm node-red
   ```
3. Execute o Node-RED:
   ```bash
   node-red
   ```
4. Acesse no navegador: [http://localhost:1880](http://localhost:1880).

---

## 📟 Nosso projeto no Wokwi
Caso você não queira configurar o projeto do zero aqui está ele feito: 
https://wokwi.com/projects/410551824578875393

## 📦 Configuração no Wokwi

### 1. Criar o Projeto

1. Acesse [Wokwi](https://wokwi.com) e clique em **New Project**.
2. Escolha **ESP32** e adicione:
   - **DHT22** (sensor de temperatura e umidade)
   - **LDR** (sensor de luminosidade)
   - **Resistor** de 10K Ohms para o LDR.

### 2. Conectar os Componentes

1. **LDR**:
   - **Uma extremidade** → **3.3V**
   - **Outra extremidade** → **GPIO 35** (A0), com resistor para **GND**.

2. **DHT22**:
   - **VCC** → **3.3V**
   - **GND** → **GND**
   - **Data** → **GPIO 23**

  ### No final é para ficar com as conexões assim
  ![image](https://github.com/user-attachments/assets/5f8f338a-ffa0-467a-b88c-edefdfad9bd5)


---

## 🚀 Código Completo no Wokwi

Cole este código no editor do Wokwi:

```cpp
// Bibliotecas
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHTesp.h>

// Rede de conexão
const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "4.228.58.205";

// Pinos
const int DHT_PIN = 23;  // Pino do DHT22
const int LDR_PIN = 35;  // Pino do LDR

// Objeto para o DHT22
DHTesp dhtSensor;

// Cliente Wi-Fi e MQTT
WiFiClient WOKWI_client;
PubSubClient client(WOKWI_client);

// Configuração do Wi-Fi
void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Conectando à rede: ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi conectado");
  Serial.print("Endereço IP: ");
  Serial.println(WiFi.localIP());
}

// Reconectar ao MQTT
void reconnect() {
  while (!client.connected()) {
    Serial.print("Tentando conexão MQTT...");
    if (client.connect("ESP32_Client")) {
      Serial.println("Conectado!");
    } else {
      Serial.print("Falhou, rc=");
      Serial.print(client.state());
      Serial.println(" Tentando novamente em 5 segundos...");
      delay(5000);
    }
  }
}

// Função para ler luminosidade
void luminosity_dados() {
  int ldrValue = analogRead(LDR_PIN);  // Lê valor analógico
  int mappedLdrValue = map(ldrValue, 0, 4095, 0, 100);  // Mapeia para 0-100
  client.publish("Azure/Luminosity", String(mappedLdrValue).c_str());  // Publica
  delay(1000);
}

// Função para ler temperatura e umidade
void DHT_dados() {
  TempAndHumidity data = dhtSensor.getTempAndHumidity();
  client.publish("Azure/Temperature", String(data.temperature).c_str());  // Temperatura
  client.publish("Azure/Humidity", String(data.humidity).c_str());        // Umidade
  delay(1000);
}

// Configuração inicial
void setup() {
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);  // Configura o DHT22
}

// Loop principal
void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  DHT_dados();        // Envia dados do DHT22
  luminosity_dados();  // Envia dados do LDR
}
```

---

## 🧑‍💻 Como Executar no Wokwi

1. Cole o código no editor do Wokwi.
2. Clique em **Start Simulation** para iniciar.
3. Acompanhe as mensagens no **Serial Monitor**.
4. No **Node-RED**, adicione um **nó MQTT IN** para cada tópico:
   - `Azure/Temperature`
   - `Azure/Humidity`
   - `Azure/Luminosity`

---

## 🛡️ Tratamento de Erros

- **Wi-Fi**: O ESP32 tentará se reconectar automaticamente.
- **MQTT**: A função `reconnect()` tentará restabelecer a conexão ao broker.

---

## 📋 Conclusão

Com este projeto, você pode testar a coleta de dados de sensores em um ambiente simulado usando o Wokwi e visualizar os dados no Node-RED via MQTT. Isso facilita o desenvolvimento de projetos IoT e automações.

