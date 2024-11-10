
# 🛰️ Projeto de Radar Ultrassônico com Arduino

Este projeto consiste em um radar ultrassônico que exibe em tempo real a distância de objetos detectados em uma interface visual estilo **sonar**. Utilizamos um **Arduino** com um **sensor HC-SR04** para capturar a distância e transmitimos os dados para uma página web usando **WebSocket**.

## 🖼️ Demonstração

![Exemplo do radar em funcionamento](https://via.placeholder.com/800x400)  
*Radar em funcionamento mostrando distância de objetos detectados*

## 🛠️ Componentes Necessários

- **Arduino Uno** ou similar
- **Sensor Ultrassônico HC-SR04**
- **Jumpers** (fios de conexão)
- **Cabo USB** para conectar o Arduino ao computador

## 🚀 Começando

### 1. Configuração do Arduino e Sensor

1. Conecte o **sensor ultrassônico HC-SR04** ao Arduino:
   - VCC -> 5V
   - GND -> GND
   - TRIG -> Pino 9
   - ECHO -> Pino 10

2. Carregue o código no Arduino. Este código fará leituras de distância e enviará as informações pela porta serial.

```cpp
// Código Arduino para leitura de distância
const int trigPin = 9;
const int echoPin = 10;

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  Serial.begin(9600);
}

void loop() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH);
  int distance = duration * 0.034 / 2;

  Serial.println(distance);
  delay(100);
}
```

### 2. Configuração do Servidor Python

Utilize Python para transmitir dados do Arduino para a página web em tempo real via WebSocket.

1. **Instale as dependências**:
   ```bash
   pip install pyserial websockets
   ```

2. **Execute o servidor** com o código abaixo, substituindo `'COM3'` pela porta correta do seu Arduino.

```python
import serial
import asyncio
import websockets

ser = serial.Serial('COM3', 9600)

async def send_distance(websocket, path):
    while True:
        if ser.in_waiting > 0:
            distance = ser.readline().decode().strip()
            await websocket.send(distance)
        await asyncio.sleep(0.1)

start_server = websockets.serve(send_distance, "localhost", 6789)
asyncio.get_event_loop().run_until_complete(start_server)
asyncio.get_event_loop().run_forever()
```

### 3. Interface Web (Radar Sonar)

1. Crie um arquivo `index.html` com o seguinte código para visualizar o radar em tempo real.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Radar Ultrassônico</title>
    <style>
        body { display: flex; justify-content: center; align-items: center; height: 100vh; background: #222; color: #fff; }
        canvas { border: 1px solid #fff; background: #111; }
    </style>
</head>
<body>
    <canvas id="sonarCanvas" width="400" height="400"></canvas>

    <script>
        const canvas = document.getElementById('sonarCanvas');
        const ctx = canvas.getContext('2d');
        const centerX = canvas.width / 2;
        const centerY = canvas.height;
        const maxDistance = 200;

        function drawSonar(distance) {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.beginPath();
            ctx.arc(centerX, centerY, 180, Math.PI, 2 * Math.PI);
            ctx.strokeStyle = '#00ff00';
            ctx.stroke();
            const angle = Date.now() / 1000 % Math.PI;
            ctx.beginPath();
            ctx.moveTo(centerX, centerY);
            ctx.lineTo(centerX + Math.cos(angle) * 180, centerY - Math.sin(angle) * 180);
            ctx.stroke();
            const distanceRatio = Math.min(distance / maxDistance, 1);
            const pointX = centerX + Math.cos(angle) * distanceRatio * 180;
            const pointY = centerY - Math.sin(angle) * distanceRatio * 180;
            ctx.beginPath();
            ctx.arc(pointX, pointY, 5, 0, 2 * Math.PI);
            ctx.fillStyle = '#ff0000';
            ctx.fill();
        }

        const ws = new WebSocket('ws://localhost:6789');
        ws.onmessage = (event) => {
            const distance = parseInt(event.data);
            drawSonar(distance);
        };
    </script>
</body>
</html>
```

## 🏃 Executando o Projeto

1. Conecte o Arduino e carregue o código.
2. Execute o servidor Python:
   ```bash
   python server.py
   ```
3. Abra `index.html` no navegador para ver o radar em tempo real.

## 📷 Imagens do Projeto

### Conexão Arduino com Sensor
![Conexão física do Arduino com sensor](https://via.placeholder.com/400x300)

### Exemplo do Radar em Funcionamento
![Interface de radar sonar em funcionamento](https://via.placeholder.com/400x300)

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## 🤝 Contribuições

Contribuições são bem-vindas! Sinta-se à vontade para fazer um fork do repositório e enviar pull requests com melhorias.
