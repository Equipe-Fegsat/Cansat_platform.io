# CanSat Sumé – FegSat

Firmware do **CanSat Sumé**, desenvolvido pela equipe FegSat (Unesp Guaratinguetá) originalmente para a **Lasc 2026**, com foco em **monitoramento climatológico**.

O firmware roda em um microcontrolador **STM32F407**, lê sensores atmosféricos e inerciais e envia os dados pela UART. A comunicação sem fio e a localização do CanSat são feitas por um **módulo LoRa**.

## Visão geral

```
 ┌──────────┐  I2C   ┌───────────────┐  UART2   ┌────────────┐
 │  BME280  │ ─────► │               │ ───────► │  Terminal  │
 ├──────────┤        │  STM32F407    │ 115200   │  (debug)   │
 │ MPU6050  │ ─────► │               │          └────────────┘
 └──────────┘        │               │   LoRa   ┌────────────┐
                     │               │ ◄──────► │ Estação de │
                     └───────────────┘          │   solo     │
                                                └────────────┘
```

## Hardware

| Componente | Função | Interface |
|---|---|---|
| **STM32F407** | Microcontrolador principal | – |
| **BME280** | Temperatura, pressão e umidade | I2C |
| **MPU6050** | Acelerômetro e giroscópio (3 eixos) + temperatura | I2C1 |
| **Módulo LoRa** | Comunicação e localização do CanSat | [SPI / UART – confirmar] |
| **USART2** | Saída serial de depuração | UART, 115200 baud |

### Pinagem

| Função | Pino |
|---|---|
| I2C1 SCL / SDA | [preencher, conforme o `.ioc`] |
| USART2 TX / RX | [preencher] |
| LoRa (SPI/UART, NSS, RESET, DIO0) | [preencher] |

## Funcionamento atual do firmware

O `main.c` foi gerado com o **STM32CubeMX** (HAL) e, depois da inicialização dos periféricos, executa:

1. **Inicialização dos sensores**
   - BME280 configurado com oversampling ×1 para temperatura, pressão e umidade, modo normal, tempo de standby de 250 ms e filtro IIR de coeficiente 2.
   - MPU6050 inicializado no barramento `hi2c1`.
   - Em caso de falha, uma mensagem de erro é enviada pela UART.
2. **Loop principal (a cada 500 ms)**
   - Lê temperatura, pressão e umidade do BME280.
   - Lê os dados do MPU6050 e converte os valores brutos para unidades físicas (g, °/s e °C).
   - Envia tudo em texto pela USART2.

### Exemplo de saída serial

```
Temp: 25.30°C | Press: 1013.25 hPa | Hum: 48.50%
[MPU6050]  Accel[g]: X=0.012 Y=-0.003 Z=0.998 | Gyro[dps]: X=0.15 Y=-0.08 Z=0.02 | Temp: 26.1 C
```

### Configuração do microcontrolador

| Item | Valor |
|---|---|
| Fonte de clock | HSI interno (sem PLL) |
| I2C1 | 100 kHz, endereçamento de 7 bits |
| USART2 | 115200 baud, 8N1 |
| SPI1 e SPI2 | Inicializados como mestre (ainda não usados no `main.c` atual) |
| I2C2 | Inicializado (ainda não usado no `main.c` atual) |

## Estrutura do projeto

```
.
├── README.md
├── Core/
│   ├── Inc/            # Headers (main.h, BME280_STM32.h, MPU6050.h, ...)
│   └── Src/            # main.c e drivers dos sensores
├── Drivers/            # HAL / CMSIS (gerados pelo CubeMX)
└── [nome_do_projeto].ioc
```

> Ajuste a árvore acima para refletir a estrutura real do seu projeto no STM32CubeIDE.

## Como compilar e gravar

1. Instale o [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html).
2. Importe o projeto (**File → Import → Existing Projects into Workspace**).
3. Conecte a placa por ST-Link e clique em **Run** (ou **Debug**).
4. Abra um terminal serial (PuTTY, CoolTerm, o monitor do CubeIDE etc.) a **115200 baud** para acompanhar as leituras.

## Próximos passos

- [ ] Integrar o módulo LoRa (telemetria e localização)
- [ ] [Outros itens, como armazenamento em cartão SD ou estrutura do pacote de telemetria]

## Equipe

Projeto desenvolvido pela **FegSat**, equipe de nanossatélites da Unesp Guaratinguetá.
