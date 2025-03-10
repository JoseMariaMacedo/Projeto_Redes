# Documentação do Código de Simulação de Redes TCP em C#

## Visão Geral

Este código simula o funcionamento de uma rede TCP utilizando um modelo de camadas. Cada camada possui sua própria estrutura de PDU (Protocol Data Unit) e os pacotes são processados ao longo das camadas até chegarem ao destino.

O código implementa:

- PDUs para diferentes camadas: Interface, Rede, Transporte e Aplicação.
- Classe `No`, que representa um dispositivo na rede.
- Classe `MotorSimulacao`, que gerencia eventos de transmissão e processamento.
- Protocolos de Transporte A e B, um com confirmação de recebimento e outro sem.
- Uma aplicação de exemplo, que usa os protocolos para enviar mensagens entre dois dispositivos.

---

## 1. Definição das PDUs (Protocol Data Units)

### 1.1 Classe PDU

```csharp
public abstract class PDU
{
    public byte[] Dados { get; set; }
}
```

Esta é a classe base para todas as PDUs, contendo apenas um campo `Dados` para armazenar os dados transmitidos.

### 1.2 PDUs Específicas

Cada classe abaixo estende `PDU` e adiciona informações específicas de cada camada:

- **`InterfacePDU` (Camada de Interface)** - Armazena endereços MAC de origem e destino.
- **`RedePDU` (Camada de Rede)** - Contém endereços lógicos e protocolo de transporte.
- **`TransportePDU` (Camada de Transporte)** - Define portas e se requer confirmação de recebimento (ACK).
- **`AplicacaoPDU` (Camada de Aplicação)** - Apenas armazena dados da aplicação.

---

## 2. Classe `No` (Dispositivo da Rede)

A classe `No` representa um dispositivo na rede, como um computador ou roteador.

### 2.1 Atributos

```csharp
public string Nome { get; set; }
public string EnderecoMac { get; set; }
public string EnderecoLogico { get; set; }
```

Cada `No` possui um nome, um endereço MAC e um endereço IP.

### 2.2 Processamento de Camadas

O `No` possui métodos para processar pacotes recebidos:

- **`ProcessarCamadaInterface`**: Verifica MACs e extrai PDU de Rede.
- **`ProcessarCamadaRede`**: Verifica endereços IPs e extrai PDU de Transporte.
- **`ProcessarCamadaTransporte`**: Verifica portas e extrai PDU de Aplicação.
- **`ProcessarCamadaAplicacao`**: Converte dados da aplicação para texto e exibe.

---

## 3. Classe `MotorSimulacao`

Esta classe gerencia os eventos e filas de transmissão. Possui quatro filas:

```csharp
private List<Evento> filaInterface = new List<Evento>();
private List<Evento> filaRede = new List<Evento>();
private List<Evento> filaTransporte = new List<Evento>();
private List<Evento> filaAplicacao = new List<Evento>();
```

Os eventos passam por essas filas até serem processados pelo `No` de destino.

### 3.1 Método `Executar`

Este método gerencia a execução dos eventos enquanto houver pacotes nas filas.

```csharp
while (continuar) {
    if (filaInterface.Count > 0) { ... }
    else if (filaRede.Count > 0) { ... }
    else if (filaTransporte.Count > 0) { ... }
    else if (filaAplicacao.Count > 0) { ... }
    else { continuar = false; }
}
```

Cada evento é retirado da fila e processado pelo `No` responsável.

---

## 4. Protocolos de Transporte

Dois protocolos de transporte são implementados:

### 4.1 `ProtocoloTransporteA` (Com Confirmação de Recebimento)

- Envia um pacote e aguarda ACK.
- Se `RequereAck` for `true`, responde com um ACK.

### 4.2 `ProtocoloTransporteB` (Sem Confirmação)

Apenas envia os dados sem esperar resposta.

```csharp
public override void Enviar(No origem, No destino, byte[] dados, int portaOrigem, int portaDestino) {
    // Cria PDUs e adiciona à fila de eventos
    Motor.AdicionarEventoInterface(destino, interfacePdu, origem);
}
```

---

## 5. Simulação

A classe `Simulacao` é o ponto de entrada do programa.

### 5.1 Criando Elementos

```csharp
var motor = new MotorSimulacao();
var computador1 = new No { Nome = "Computador1", EnderecoMac = "00:11:22:33:44:55", EnderecoLogico = "192.168.0.1" };
var computador2 = new No { Nome = "Computador2", EnderecoMac = "AA:BB:CC:DD:EE:FF", EnderecoLogico = "192.168.0.2" };
```

### 5.2 Criando Aplicativos

Cada computador tem uma aplicação associada a um protocolo de transporte.

```csharp
var app1 = new AplicacaoExemplo(computador1, protocoloA);
var app2 = new AplicacaoExemplo(computador2, protocoloB);
```

### 5.3 Enviando Mensagens

```csharp
app1.EnviarMensagem(computador2, "Olá, mundo!", 1234, 80);
app2.EnviarMensagem(computador1, "Resposta sem confirmação.", 80, 1234);
```

### 5.4 Executando a Simulação

```csharp
motor.Executar();
```

A simulação processa todos os eventos até esvaziar as filas.

---

## Conclusão

Este código implementa uma simulação simplificada de transmissão de dados em redes TCP, com protocolos de transporte diferentes. Cada camada processa os pacotes, garantindo que os dados sejam transmitidos corretamente de origem a destino.
