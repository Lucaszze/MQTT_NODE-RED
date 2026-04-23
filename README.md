# 🏠 Automação Residencial com MQTT, HiveMQ e Node-RED

Projeto de simulação de automação residencial utilizando protocolo MQTT, broker HiveMQ, GitHub Codespace e Node-RED via FlowFuse.

---

## 📐 Arquitetura

```
[Codespace - Mosquitto]         [HiveMQ Broker]         [FlowFuse - Node-RED]
  mosquitto_pub           →    broker.hivemq.com    →    Subscreve tópicos
  simulador.py            →    (broker público)     →    Processa dados
  mosquitto_sub           ←                         ←    Publica comandos
                                                              ↓
                                                         [Dashboard /ui]
                                                         Gauge de brilho
                                                         Status da lâmpada
                                                         Botão de controle
```

### Decisões de arquitetura

**Por que MQTT e não HTTP?**
O protocolo MQTT é leve, assíncrono e ideal para dispositivos IoT. Diferente do HTTP (requisição-resposta), o MQTT usa o modelo publish/subscribe, permitindo que múltiplos clientes publiquem e consumam dados simultaneamente sem acoplamento direto.

**Por que HiveMQ público?**
O `broker.hivemq.com` é um broker MQTT gratuito e público, eliminando a necessidade de hospedar infraestrutura própria. Ideal para prototipação e trabalhos acadêmicos.

**Por que separar `/status` e `/cmd`?**
A separação de tópicos garante controle bidirecional claro:
- `/status` — fluxo de dados dos dispositivos para o sistema
- `/cmd` — fluxo de comandos do sistema para os dispositivos

Isso evita loops e facilita o debug.

---

## 📡 Tópicos MQTT

| Tópico | Direção | Payload |
|---|---|---|
| `casa/sala/lampada/status` | Codespace → Node-RED | `{"ligada": true, "brilho": 80}` |
| `casa/sala/tomada/status` | Codespace → Node-RED | `{"ligada": false, "consumo_w": 0}` |
| `casa/sala/lampada/cmd` | Node-RED → Codespace | `{"acao": "ligar"}` |
| `casa/sala/tomada/cmd` | Node-RED → Codespace | `{"acao": "desligar"}` |

---

## 🧰 Tecnologias utilizadas

| Ferramenta | Função |
|---|---|
| GitHub Codespace | Ambiente de desenvolvimento e simulação |
| Mosquitto Clients | Cliente MQTT via terminal |
| Python + paho-mqtt | Simulador de sensores |
| HiveMQ | Broker MQTT público |
| FlowFuse | Node-RED gerenciado na nuvem |
| node-red-dashboard | Dashboard visual |

---

## 🚀 Como executar

### Pré-requisitos

- GitHub Codespace configurado
- Conta no FlowFuse com instância Node-RED rodando
- `mosquitto-clients` instalado
- `paho-mqtt` instalado

### 1. Instalar dependências no Codespace

```bash
sudo apt-get update && sudo apt-get install -y mosquitto-clients
pip install paho-mqtt
```

### 2. Rodar o simulador Python

```bash
python simulador.py
```

O simulador publica dados aleatórios a cada 3 segundos nos tópicos de status.

### 3. Publicar manualmente via Mosquitto

```bash
# Publicar status da lâmpada
mosquitto_pub -h broker.hivemq.com -p 1883 \
  -t "casa/sala/lampada/status" \
  -m '{"ligada": true, "brilho": 85}'

# Enviar comando de controle
mosquitto_pub -h broker.hivemq.com -p 1883 \
  -t "casa/sala/lampada/cmd" \
  -m '{"acao": "ligar"}'

# Escutar todos os tópicos da casa
mosquitto_sub -h broker.hivemq.com -p 1883 -t "casa/sala/#"
```

### 4. Acessar o dashboard

```
https://team-vinicius-santarelli-vsantarellivms-f4d3-c7a32f1d.flowfuse.cloud/ui
```

### 5. Publicar via HiveMQ Web Client

Acesse `http://www.hivemq.com/demos/websocket-client/` e conecte com:
- **Host:** `broker.hivemq.com`
- **Port:** `8884`
- **SSL:** ativado

---

## 🔄 Fluxo Node-RED

```
[mqtt in]  →  [json]  →  [change: brilho]  →  [ui_gauge]
                      →  [change: ligada]  →  [ui_text]

[ui_button]  →  [function: formata cmd]  →  [mqtt out]
```

### Nós utilizados

- **mqtt in** — subscreve `casa/sala/lampada/status`
- **json** — converte payload string para objeto
- **change** — extrai campos individuais (`brilho`, `ligada`)
- **ui_gauge** — exibe brilho em gauge de 0–100%
- **ui_text** — exibe status ligada/desligada
- **ui_button** — envia comando de ligar
- **function** — formata payload do comando
- **mqtt out** — publica em `casa/sala/lampada/cmd`

---

## 📊 Demonstração prática

### Dados chegando
Com o simulador rodando, o dashboard atualiza automaticamente o gauge de brilho e o status da lâmpada a cada 3 segundos.

### Controle sendo enviado
Ao clicar em **"Ligar Lâmpada"** no dashboard, o Node-RED publica `{"acao": "ligar"}` no tópico de comando. O Codespace recebe via `mosquitto_sub` ou pelo simulador Python.

### Duas origens de dados
- **Via Codespace (Mosquitto):** `mosquitto_pub` direto no terminal
- **Via HiveMQ Web Client:** interface web do broker publicando no mesmo tópico

---

## 📁 Arquivos

```
/
├── simulador.py       # Simulador de sensores em Python
└── README.md          # Este arquivo
```

---

## 👤 Autor

Vinicius Santarelli — Abril 2026
