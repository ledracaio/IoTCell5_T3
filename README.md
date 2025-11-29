[![Desenvolvido por Caio Augusto Ledra](https://img.shields.io/badge/Desenvolvido%20por-Caio%20Augusto%20Ledra-007ACC?style=flat-square)](https://github.com/ledracaio)  [![Plataforma: Node-RED](https://img.shields.io/badge/Plataforma-Node--RED-B22222?style=flat-square&logo=nodered)](https://nodered.org/)  [![Banco de Dados: MongoDB](https://img.shields.io/badge/Banco%20de%20Dados-MongoDB-47A248?style=flat-square&logo=mongodb)](https://www.mongodb.com/)    [![Ambiente: Docker Compose](https://img.shields.io/badge/Ambiente-Docker%20Compose-2496ED?style=flat-square&logo=docker)](https://docs.docker.com/compose/)  [![Curso: UNIDAVI SI](https://img.shields.io/badge/Curso-SI%20UNIDAVI-blue?style=flat-square)]()

---

# 🏭 Célula 5 – Contagem e Rastreamento de Itens na Linha

## Trabalho 03 – Integração com Node-RED, Dashboard e Banco de Dados

### Curso de Sistemas de Informação – UNIDAVI

**Aluno:** Caio Augusto Ledra  
**Professor:** Esp. Ademar Perfoll Junior  
**Disciplina:** Internet das Coisas (IoT)

---

# 📘 Descrição do Projeto

Este trabalho corresponde à **etapa final da Célula 5**, integrando todo o sistema de contagem e rastreamento de itens à plataforma IoT utilizando:

* **Node-RED**
* **Dashboard Web**
* **Banco de Dados MongoDB**
* **Broker MQTT Mosquitto**
* **Docker Compose**
* **Dispositivo ESP8266 (Trabalho 2)**

O objetivo é transformar a célula física em uma **plataforma IoT completa**, com telemetria em tempo real, visualização histórica, eventos de operação e interação remota via comandos MQTT.

---

# 🧠 Objetivos do Trabalho 03

* Criar e disponibilizar um **flow completo no Node-RED**.  
* Processar telemetria e eventos via MQTT.  
* Validar e normalizar pacotes JSON no Node-RED.  
* Salvar telemetria e eventos no **MongoDB**.  
* Construir um **dashboard interativo** com:  
  * indicadores em tempo real  
  * status colorido  
  * gráfico histórico  
  * tabela de eventos  
* Implementar botões de ação (`get_status`).  
* Usar **Docker Compose para toda a infraestrutura IoT**.  
* Demonstrar evidências do funcionamento do sistema.  

---

# 🐳 Infraestrutura IoT (Docker Compose)

## 📦 Arquitetura dos Containers

O ambiente roda completamente em Docker, conforme especificado pelo professor:

* `Node-RED` → fluxo + dashboard  
* `MongoDB` → armazenamento de histórico  
* `Mongo-Express` → interface web do MongoDB  
* `Mosquitto` → broker MQTT que se comunica com o ESP8266  

---

## 📝 docker-compose.yml (versão final usada)

```yaml
services:
  nodered:
    image: nodered/node-red:latest
    container_name: nodered
    ports:
      - "1880:1880"
    environment:
      - TZ=America/Sao_Paulo
    volumes:
      - nodered_data:/data
    depends_on: [ mongo, mosquitto ]
    networks: [ iot ]

  mongo:
    image: mongo:6
    container_name: mongo
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=admin123
    volumes:
      - mongo_data:/data/db
    networks: [ iot ]

  mongo-express:
    image: mongo-express:1
    container_name: mongo-express
    ports:
      - "8081:8081"
    environment:
      - ME_CONFIG_MONGODB_URL=mongodb://admin:admin123@mongo:27017/?authSource=admin
      - ME_CONFIG_BASICAUTH=true
      - ME_CONFIG_BASICAUTH_USERNAME=admin
      - ME_CONFIG_BASICAUTH_PASSWORD=admin123
    depends_on: [ mongo ]
    networks: [ iot ]

  mosquitto:
    image: eclipse-mosquitto:2
    container_name: mosquitto
    ports:
      - "1884:1883"
    volumes:
      - ./mosquitto/config:/mosquitto/config:ro
      - mosquitto_data:/mosquitto/data
      - mosquitto_log:/mosquitto/log
    networks: [ iot ]

volumes:
  nodered_data:
  mongo_data:
  mosquitto_data:
  mosquitto_log:

networks:
  iot:
    driver: bridge
```

---

# 🚀 Como Subir o Ambiente

### 1️⃣ Criar a pasta do projeto
```
mkdir celula5
cd celula5
```

### 2️⃣ Criar o arquivo `docker-compose.yml` (colar o conteúdo acima)

### 3️⃣ Criar a pasta de config do Mosquitto
```
mkdir -p mosquitto/config
```

### 4️⃣ Criar o arquivo de configuração do Mosquitto  
`mosquitto/config/mosquitto.conf`:

```
listener 1883
allow_anonymous true
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
```

### 5️⃣ Subir todos os serviços
```
docker compose up -d
```

---

# 🌐 Serviços Disponíveis

| Serviço              | URL                                                  | Função                 |
| -------------------- | ---------------------------------------------------- | ---------------------- |
| **Node-RED**         | [http://localhost:1880](http://localhost:1880)       | Dashboard + Flow       |
| **Dashboard UI**     | [http://localhost:1880/ui](http://localhost:1880/ui) | Monitoramento completo |
| **Mongo Express**    | [http://localhost:8081](http://localhost:8081)       | Interface do banco     |
| **Mosquitto (MQTT)** | tcp://localhost:1884                                 | Broker MQTT local      |

---

# 📡 Estrutura de Tópicos MQTT (mantém o padrão do Trabalho 2)

```
iot/riodosul/si/BSN22025T26F8/cell/5/device/c5-caio-ledra/
```

| Tipo      | Tópico        | Direção |
| --------- | ------------- | ------- |
| State     | .../state     | Recebe  |
| Telemetry | .../telemetry | Recebe  |
| Event     | .../event     | Recebe  |
| Cmd       | .../cmd       | Envia   |

---

# 📝 Importação do Flow Node-RED

1. Acesse **[http://localhost:1880](http://localhost:1880)**  
2. Menu → **Import**  
3. Cole o arquivo **flows.json** entregue  
4. Clique **Import**  
5. Clique **Deploy**  

---

# 🧩 Estrutura Interna do Flow

O flow contém:

- **MQTT-in (telemetry/event/state)** → Recebem dados do ESP8266  
- **Validação (function)** → Garante que o payload contém `ts`, `cellId`, `devId`, `metrics`, `status`  
- **Normalização (function)** → Insere `cellId/devId` do tópico caso não venham no JSON  
- **MongoDB Out** → Grava telemetria e eventos  
- **Dashboard UI** → Widgets exibem contador, velocidade, status colorido, gráfico histórico, tabela de eventos, indicador online/offline  
- **MQTT-out (commands)** → Envia:

```json
{"action":"get_status"}
```

---

# 📊 Dashboard – Componentes Finalizados

* Indicador Online/Offline  
* Card de status colorido  
* Gauges (velocidade / contador)  
* Métricas em texto  
* Gráfico de telemetria (MongoDB)  
* Tabela de eventos recentes  
* Botão GET STATUS  

Todas as figuras estão no relatório.

---

# 🗄 Estrutura de Banco de Dados (MongoDB)

### 📁 telemetry
Armazena: `ts`, `metrics`, `status`, `units`

### 📁 event
Armazena: `ts`, `type`, `info`

---

# 💬 Testes Realizados

| Teste                     | Resultado |
| ------------------------- | --------- |
| Estado online/offline     | OK        |
| Recepção de telemetria    | OK        |
| Recepção de eventos       | OK        |
| Botão get_status          | OK        |
| Gráfico histórico         | OK        |
| Tabela de eventos         | OK        |
| Reconexão MQTT automática | OK        |

---

# 🧩 Comando de Teste MQTT

```
mosquitto_pub -h localhost -p 1884 \
  -t "iot/.../cmd" \
  -m '{"action":"get_status"}'
```

---

# 🧷 Referências

- Documentação Node-RED: https://nodered.org/docs/  
- Docker Compose: https://docs.docker.com/compose/  
- Mosquitto MQTT: https://mosquitto.org/  
- MongoDB: https://www.mongodb.com/  

---

# Desenvolvido por @ledracaio
