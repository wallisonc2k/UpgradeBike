# 🚲🔊 Projeto: Bicicleta Infantil com Som de Moto e Acelerador Realista

## 📌 Objetivo

Transformar uma bicicleta comum infantil em uma bicicleta interativa com:

- 🎮 Acelerador no guidão
- 🔊 Som de motor de moto realista
- 📢 Buzina
- 🔋 Alimentação por powerbank (simples e recarregável)
- 🧠 Controle inteligente via ESP32

> O projeto **não utiliza motor elétrico nem tração**.
> O acelerador serve apenas para controlar os efeitos sonoros.

---

## 🎯 Conceito Principal

O sistema utilizará:

- Um acelerador eletrônico de bicicleta/patinete
- Um microcontrolador ESP32
- Reprodução contínua de áudio
- Alteração dinâmica do som conforme o acelerador

Diferente de projetos simples que apenas trocam músicas, este projeto utilizará o **Método de Som Contínuo em Loop**:

O som do motor ficará reproduzindo continuamente, enquanto o sistema altera:

- Intensidade
- Volume
- Rotação simulada
- Velocidade do loop
- Pitch (opcional)

Isso cria um efeito muito mais realista.

---

## 🧩 Componentes Necessários

### 🧠 ESP32 — Controle Principal

Responsável por:

- Leitura do acelerador
- Controle do áudio
- Lógica do sistema

Características:

- ADC integrado
- PWM
- Alta velocidade
- Bluetooth (expansão futura)

---

### 🎮 Acelerador

Pode ser usado:

- Acelerador de polegar (Hall Effect)
- Meia manopla
- Acelerador Hall
- Potenciômetro adaptado

**Função:** controlar a intensidade do som do motor.

---

### 🔊 Sistema de Áudio — MAX98357A + ESP32

O **MAX98357A** será usado como amplificador de áudio digital via I2S.

Ele:

- Recebe áudio digital do ESP32
- Converte para som
- Amplifica para o alto-falante

> ⚠️ **Importante:**
> - O MAX98357A **não armazena áudio**
> - Ele **não lê microSD**
> - Ele **não decodifica MP3 sozinho**
>
> Quem faz o controle do áudio é o **ESP32**.

---

### 💾 Armazenamento dos Áudios — Módulo microSD SPI

Os arquivos de áudio ficarão no cartão microSD, acessado pelo ESP32 por SPI.

Função do módulo:

- Permitir leitura dos arquivos de áudio
- Armazenar os loops do motor e a buzina
- Manter o projeto flexível e fácil de atualizar

**Arquitetura correta:**

```
microSD → ESP32 → MAX98357A → alto-falante
```

---

### 🔈 Alto-falante

Sugestão:

- 4Ω
- 3W ou 5W

Preferencialmente com caixa acústica pequena e resistente a vibração.

---

### 📢 Buzina

Botão independente instalado no guidão. Ao pressionar, reproduz o arquivo de buzina.

---

### 🔋 Alimentação — Powerbank USB

**Opção recomendada — Powerbank 5V USB**

Esta é a forma mais simples, segura e prática de alimentar o projeto.

#### Por que powerbank?

| Vantagem | Detalhe |
|---|---|
| ✅ Plug and play | Conecta direto no ESP32 via USB — sem circuito extra |
| ✅ Recarregável | Qualquer carregador USB ou USB-C |
| ✅ Proteção nativa | Já tem proteção contra sobrecarga, curto e descarga profunda |
| ✅ Seguro para crianças | Sem baterias expostas ou módulos de carga soltos |
| ✅ Autonomia excelente | 5000mAh → 6 a 8 horas de uso |
| ✅ Fácil de substituir | Troca sem abrir a caixa eletrônica |

#### Especificações recomendadas

- Capacidade: **5000 mAh** (o suficiente para uma tarde toda de brincadeira)
- Saída: **5V / 1A ou 2A** — padrão USB-A
- Tamanho: modelo compacto (slim), mais fácil de fixar no quadro
- Opcional: modelo com passthrough (funciona enquanto carrega)

#### Como conectar

```
Powerbank (USB-A) → Cabo USB-A para Micro-USB → ESP32 DevKit V1
```

O ESP32 recebe 5V pelo pino USB e distribui 3.3V regulado para os módulos (microSD, acelerador).
O MAX98357A pode ser alimentado pelo pino 5V (VIN) do ESP32.

#### Fixação na bicicleta

- Use velcro dupla-face adesivo para prender o powerbank ao quadro
- Posicione perto da caixa eletrônica para minimizar o comprimento do cabo USB
- Um elástico de bike ou abraçadeira de nylon adiciona segurança extra

---

## ⚙️ Funcionamento do Sistema

### 🎮 Acelerador

O acelerador envia um valor analógico ao ESP32:

| Posição | Valor ADC |
|---|---|
| Solto (idle) | ~800 |
| Aceleração leve | ~1500 |
| Aceleração média | ~2500 |
| Totalmente pressionado | ~3800 |

---

### 🔊 Motor em Loop

O sistema mantém um áudio contínuo reproduzindo (ex: `idle.wav`).

O ESP32 controla dinamicamente:

- Volume
- Ganho
- Velocidade de reprodução
- Troca suave entre camadas de áudio

---

### 🎚️ Estratégia de Áudio — Método Híbrido em Camadas

Utilizar múltiplos loops:

| Arquivo | Situação |
|---|---|
| `idle.wav` | Motor em marcha lenta |
| `low.wav` | Aceleração leve |
| `medium.wav` | Aceleração média |
| `high.wav` | Aceleração máxima |

O ESP32 alterna suavemente entre eles conforme a posição do acelerador.

---

### 🔁 Fluxo de Funcionamento

1. **Estado inicial** → motor em marcha lenta
2. **Acelerador leve** → som aumenta suavemente
3. **Aceleração máxima** → som agressivo e alto
4. **Soltou acelerador** → desaceleração gradual

---

## 📁 Estrutura do Cartão SD

```
/audio
 ├── 0001.wav  → idle
 ├── 0002.wav  → low
 ├── 0003.wav  → medium
 ├── 0004.wav  → high
 ├── 0005.wav  → buzina
 ├── 0006.wav  → partida
 └── 0007.wav  → desaceleração
```

> Formato: **WAV 16-bit PCM, Mono, 22050 Hz**

---

## 🔌 Ligações

### microSD → ESP32 via SPI

| microSD | ESP32 |
|---|---|
| VCC | 3.3V |
| GND | GND |
| MISO | GPIO19 |
| MOSI | GPIO23 |
| SCK | GPIO18 |
| CS | GPIO5 |

### ESP32 → MAX98357A via I2S

| MAX98357A | ESP32 |
|---|---|
| VIN | 5V (pino VIN do ESP32) |
| GND | GND |
| DIN | GPIO25 |
| BCLK | GPIO27 |
| LRC | GPIO26 |

### Acelerador → ESP32

| Fio | ESP32 |
|---|---|
| VCC (vermelho) | 3.3V |
| GND (preto) | GND |
| Sinal (verde) | GPIO34 (ADC) |

### Buzina → ESP32

| Terminal | ESP32 |
|---|---|
| Terminal 1 | GPIO32 (INPUT_PULLUP) |
| Terminal 2 | GND |

### Powerbank → ESP32

```
Powerbank USB-A → Cabo USB → ESP32 (entrada Micro-USB)
```

---

## 🎯 Recomendação de Áudio

Formato ideal: **WAV 16-bit PCM, mono, 22050 Hz**

Motivos:

- Menor latência
- Melhor para loops
- Mais fácil de processar no ESP32
- Mais adequado para controle de RPM simulado

Fonte de sons gratuitos: [freesound.org](https://freesound.org) — pesquise por `motorcycle engine loop` com licença CC0.

---

## 🔧 Lista de Componentes

| Categoria | Componente | Preço estimado |
|---|---|---|
| Controle | ESP32 DevKit V1 | R$ 25–40 |
| Amplificador | MAX98357A I2S Audio Amplifier | R$ 12–20 |
| Armazenamento | Módulo microSD SPI | R$ 5–10 |
| Acelerador | Hall Effect Thumb Throttle | R$ 15–25 |
| Alto-falante | 4 Ohm 5W Speaker + caixa selada | R$ 15–30 |
| Alimentação | **Powerbank 5000mAh USB** | R$ 30–60 |
| Buzina | Botão momentâneo | R$ 2–5 |
| Fixação | Velcro dupla-face + abraçadeiras | R$ 5–10 |

**Total estimado: R$ 109–200**

> 💡 Itens removidos em relação à versão anterior: TP4056, MT3608, célula 18650 avulsa.
> O powerbank substitui tudo isso com mais segurança e praticidade.

---

## 🚀 Recursos Futuros Possíveis

O ESP32 permite expandir para:

- 🔵 **Bluetooth** — controle por celular
- 💡 **LED RGB** — simular painel
- 🔴 **Escape iluminado** — LED vermelho pulsando
- 📳 **Vibração** — motor vibracall simulando motor
- 🚴 **Sensor de movimento** — som só funciona pedalando

---

## ⚠️ Observações Importantes

### Acústica em ambiente aberto

O maior desafio não será a parte mecânica — será a **qualidade do áudio em ambiente aberto**.

Bicicleta em área externa sofre com:

- Vento
- Ruído da rua
- Vibração

Por isso:

- **Caixa acústica faz enorme diferença**
- **Posicionamento do falante importa muito**

Uma caixa pequena selada melhora bastante o resultado.

### Sobre o powerbank

- Verifique se o powerbank não desliga automaticamente com carga baixa (alguns modelos cortam após ~30 segundos sem carga suficiente). O ESP32 consome pouco — prefira modelos que **não tenham auto-desligamento**, ou use um que reconheça cargas baixas (procure por "always-on powerbank").
- Modelos com **passthrough** permitem usar enquanto carrega — prático para não perromper a brincadeira.
