# Atividade 10: Fundamentos de Escalabilidade Horizontal

### Vitor Hugo Boaventura da Silva

Versão adaptada do [tutorial de Marcus Vinícius](https://docs.google.com/document/d/10_ggD-E77lwTTbaE9z2mZoP5XwC_BCWWWRlkk72tr4k/edit?usp=sharing).

Estou considerando a utilização de bash neste tutorial. Se o seu sistema operacional for Windows, utilize o WSL.

## Introdução

Nesta atividade, você irá construir um ambiente de microserviços com foco em escalabilidade horizontal, resiliência e monitoramento, utilizando as seguintes ferramentas:

- Docker Compose
- Node.js (Express)
- RabbitMQ
- Consul
- Prometheus
- Docker Desktop

Docker Compose: Orquestra e gerencia múltiplos containers, facilitando o deploy e a comunicação entre os microserviços e suas dependências.

Node.js (Express): Plataforma e framework para criar os microserviços, responsáveis por processar requisições HTTP e interagir com a fila de mensagens.

RabbitMQ: Sistema de mensageria que permite a comunicação assíncrona entre os microserviços, garantindo desacoplamento e resiliência no processamento de pedidos.

Consul: Ferramenta de service discovery e health check, responsável por registrar automaticamente os microserviços e monitorar sua disponibilidade na infraestrutura.

Prometheus: Solução de monitoramento que coleta métricas dos microserviços, permitindo acompanhar o desempenho, disponibilidade e gerar alertas.

Docker Desktop: Interface gráfica para visualizar, iniciar, parar e gerenciar containers de forma prática no ambiente Windows.

Com esse conjunto de ferramentas, você irá experimentar conceitos fundamentais de escalabilidade horizontal, tolerância a falhas e observabilidade em sistemas distribuídos modernos.

## 1. Preparação do Ambiente

Antes de começar, instale:

1. Docker Desktop (Pode ser utilizado o Rancher Desktop, Docker via terminal ou extensão do VS Code).
2. VS Code (Para editar os arquivos).
3. Node.js (Opcional, apenas para testes locais, pois o Docker cuidará disso).

## 2. Criando a Estrutura de Pastas

Abra o console e execute o comando abaixo (ou crie as pastas manualmente):

```bash
mkdir lab-escalabilidade && cd lab-escalabilidade && mkdir users-service products-service orders-service
```

## 3. Criando os Microserviços (Código)

Você vai criar os mesmos dois arquivos dentro de cada uma das três pastas (`users-service`, `products-service` e `orders-service`).

### Arquivo A: package.json

Crie o arquivo `package.json` dentro das 3 pastas. Ele define as dependências. Utilize o comando:

```bash
for dir in users-service products-service orders-service; do
  cat > "$dir/package.json" <<EOF
{
  "name": "$dir",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "express": "^4.18.2",
    "amqplib": "^0.10.3",
    "prom-client": "^14.1.1",
    "axios": "^1.6.7"
  }
}
EOF
done
```
### Arquivo B: Dockerfile

Crie este arquivo (sem extensão) dentro das 3 pastas. Ele é a receita do container:


```bash
for dir in users-service products-service orders-service; do
  cat > "$dir/Dockerfile" <<EOF
    FROM node:18-slim
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    EXPOSE 3000
    CMD ["node", "index.js"]
EOF
done
```

### Arquivo C: index.js (O Coração)

Aqui vamos diferenciar um pouco cada serviço.

#### Em `users-service/index.js` e `products-service/index.js` (Consumidores):

```bash
for dir in users-service products-service; do
  cat > "$dir/index.js" <<EOF
    const express = require('express');
    const amqp = require('amqplib');
    const client = require('prom-client');
    const axios = require('axios');
    const app = express();

    const SERVICE_NAME = process.env.SERVICE_NAME || 'users-service';
    const SERVICE_ID = SERVICE_NAME + '-1';
    const CONSUL_HOST = process.env.CONSUL_HOST || 'consul';
    const CONSUL_PORT = process.env.CONSUL_PORT || 8500;
    const SERVICE_PORT = 3000;

    // Prometheus metrics
    client.collectDefaultMetrics();
    app.get('/metrics', async (req, res) => {
      res.set('Content-Type', client.register.contentType);
      res.end(await client.register.metrics());
    });

    // Consul registration
    async function registerConsul() {
      try {
        await axios.put(`http://${CONSUL_HOST}:${CONSUL_PORT}/v1/agent/service/register`, {
          Name: SERVICE_NAME,
          ID: SERVICE_ID,
          Address: SERVICE_NAME,
          Port: SERVICE_PORT,
          Check: {
            HTTP: `http://${SERVICE_NAME}:${SERVICE_PORT}/metrics`,
            Interval: '10s'
          }
        });
        console.log(`[CONSUL] Serviço registrado: ${SERVICE_NAME}`);
      } catch (err) {
        console.log('[CONSUL] Falha ao registrar, tentando novamente em 5s...');
        setTimeout(registerConsul, 5000);
      }
    }

    // RabbitMQ consumer
    async function initRabbit() {
      try {
        const conn = await amqp.connect('amqp://rabbitmq');
        const ch = await conn.createChannel();
        await ch.assertQueue('order_created');
        ch.consume('order_created', (msg) => {
          console.log(' [!] Evento recebido no serviço:', msg.content.toString());
          ch.ack(msg);
        });
      } catch (e) {
        console.log('Aguardando RabbitMQ...');
        setTimeout(initRabbit, 5000);
      }
    }

    app.get('/', (req, res) => res.send('Serviço Ativo'));

    app.listen(SERVICE_PORT, () => {
      console.log(`Rodando na porta ${SERVICE_PORT}`);
      initRabbit();
      registerConsul();
    });
EOF
done
```


#### Em `orders-service/index.js` (Produtor):

```bash
nano orders-service/index.js
```
e cole:

```js
const express = require('express');
const amqp = require('amqplib');
const client = require('prom-client');
const axios = require('axios');
const app = express();

const SERVICE_NAME = process.env.SERVICE_NAME || 'orders-service';
const SERVICE_ID = SERVICE_NAME + '-1';
const CONSUL_HOST = process.env.CONSUL_HOST || 'consul';
const CONSUL_PORT = process.env.CONSUL_PORT || 8500;
const SERVICE_PORT = 3000;

// Prometheus metrics
client.collectDefaultMetrics();
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});

// Consul registration
async function registerConsul() {
  try {
    await axios.put(`http://${CONSUL_HOST}:${CONSUL_PORT}/v1/agent/service/register`, {
      Name: SERVICE_NAME,
      ID: SERVICE_ID,
      Address: SERVICE_NAME,
      Port: SERVICE_PORT,
      Check: {
        HTTP: `http://${SERVICE_NAME}:${SERVICE_PORT}/metrics`,
        Interval: '10s'
      }
    });
    console.log(`[CONSUL] Serviço registrado: ${SERVICE_NAME}`);
  } catch (err) {
    console.log('[CONSUL] Falha ao registrar, tentando novamente em 5s...');
    setTimeout(registerConsul, 5000);
  }
}

app.get('/create-order', async (req, res) => {
  try {
    const conn = await amqp.connect('amqp://rabbitmq');
    const ch = await conn.createChannel();
    const msg = JSON.stringify({ id: Math.random(), item: 'Teclado', status: 'Criado' });
    ch.sendToQueue('order_created', Buffer.from(msg));
    res.send('Pedido enviado para a fila!');
  } catch (e) {
    res.status(500).send('Erro no RabbitMQ');
  }
});

app.listen(SERVICE_PORT, () => {
  console.log(`Order Service na porta ${SERVICE_PORT}`);
  registerConsul();
});
```

## 4. O Maestro: docker-compose.yml

Crie este arquivo na raiz da pasta `lab-escalabilidade`. Ele vai subir toda a infraestrutura de uma vez.

Execute o comando:

```bash
nano docker-compose.yml
```

e cole:

```yaml
version: '3.9'
services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"

  consul:
    image: consul:1.15
    ports:
      - "8500:8500"
    command: ["agent", "-dev", "-client=0.0.0.0"]

  users-service:
    build: ./users-service
    ports:
      - "3001:3000"
    depends_on:
      - rabbitmq
    environment:
      - SERVICE_NAME=users-service
      - CONSUL_HOST=consul
      - CONSUL_PORT=8500

  products-service:
    build: ./products-service
    ports:
      - "3002:3000"
    depends_on:
      - rabbitmq
    environment:
      - SERVICE_NAME=products-service
      - CONSUL_HOST=consul
      - CONSUL_PORT=8500

  orders-service:
    build: ./orders-service
    ports:
      - "3003:3000"
    depends_on:
      - rabbitmq
    environment:
      - SERVICE_NAME=orders-service
      - CONSUL_HOST=consul
      - CONSUL_PORT=8500

  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
```

## 5. Configuração do Monitoramento

Crie o arquivo `prometheus.yml` na raiz. Utilize o comando

```bash
nano prometheus.yml
```

e cole: 

```yaml
global:
  scrape_interval: 5s
scrape_configs:
  - job_name: 'microservices'
    static_configs:
      - targets: ['users-service:3000', 'products-service:3000', 'orders-service:3000']
```

## 6. Executando e Testando

- Utilize o seguinte comando para executar toda a aplicação:

```powershell
docker compose up --build
```

- Verificar os Painéis (No Navegador):

  - RabbitMQ: http://localhost:15672
    - Login: guest / Senha: guest
    - Veja a fila `order_created` sendo criada e as mensagens entrando e saindo rapidamente.
    - O gráfico de “Ready” e “Unacked” deve ficar em zero após o processamento.

  - Consul: http://localhost:8500
    - Veja os serviços `users-service`, `products-service` e `orders-service` registrados e com status “passing”.

  - Prometheus: http://localhost:9090
    - Em “Status > Targets”, veja os três microserviços como “UP”.
    - Em “Graph/Explore”, busque métricas como `process_cpu_user_seconds_total` para ver dados coletados dos serviços.

- Testar a Comunicação:

  - Acesse http://localhost:3003/create-order.
  - No terminal do Docker (ou VS Code), veja os logs dos serviços users e products imprimindo:

```
[!] Evento recebido no serviço: ...
```

## 7. Experimento de Escalabilidade (Resiliência)

Para simular uma falha e testar a tolerância a falhas:

1. Interrompa um serviço consumidor:

```powershell
docker stop lab-escalabilidade-users-service-1
```

2. Envie um pedido:

- Acesse novamente http://localhost:3003/create-order.

O resultado esperado é que a requisição seja recebida pelo outro micro-service (_products-service_).

Agora, interrompa o outro serviço com `docker stop lab-escalabilidade-products-service-1`

3. Observe:

- No RabbitMQ, a fila `order_created` terá uma mensagem “Ready” (aguardando consumidor).
- No Prometheus, o serviço parado ficará como “DOWN” em “Targets”.
- No Consul, o serviço parado ficará como “critical” ou sumirá da lista.

4. Suba os serviços consumidores novamente:

```powershell
docker start lab-escalabilidade-users-service-1
docker start lab-escalabilidade-products-service-1
```

5. Resultado esperado:

- Assim que o container subir, ele processa a mensagem pendente (veja o log no terminal).
- O RabbitMQ volta a mostrar a fila vazia.
- O status do serviço volta a “UP” no Prometheus e “passing” no Consul.

Dica: No Windows, o Docker Desktop tem uma aba chamada "Containers" onde você pode ver todos eles rodando e clicar no botão de "Stop" ou "Trash" para testar a resiliência de forma visual. Também é possível utilizar o Rancher Desktop, uma extensão do Docker para VS Code ou ainda rodar o Docker via terminal (com WSL no Windows).
