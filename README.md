<img width="1388" height="669" alt="image" src="https://github.com/user-attachments/assets/8218905e-12bd-4ba2-b67b-5d9cf82a7ba2" />
<img width="1417" height="840" alt="image" src="https://github.com/user-attachments/assets/6e8e51e8-2b6c-41d8-bbcf-ef0b4d23d4cc" />
<img width="1476" height="312" alt="image" src="https://github.com/user-attachments/assets/68e2ded2-adb9-4164-b062-43161f400bfd" />

# Node.js Microservices Platform

Este projeto é uma plataforma de microsserviços construída com Node.js, utilizando arquitetura de serviços independentes para gerenciar pedidos (orders) e faturas (invoices). A plataforma integra mensageria assíncrona, banco de dados PostgreSQL, API Gateway com Kong, e infraestrutura provisionada via Pulumi na AWS.

## Arquitetura

A arquitetura segue princípios de microsserviços, com comunicação assíncrona via RabbitMQ e exposição de APIs através de um gateway Kong. Cada serviço é containerizado com Docker e pode ser executado localmente ou implantado na nuvem.

### Serviços

- **app-orders**: Serviço responsável pela criação e gerenciamento de pedidos. Inclui:
  - API HTTP para criação de pedidos.
  - Integração com banco de dados PostgreSQL usando Drizzle ORM.
  - Publicação de mensagens no RabbitMQ quando um pedido é criado.
  - Tracing com OpenTelemetry e Jaeger.

- **app-invoices**: Serviço responsável pela geração de faturas baseadas nos pedidos. Inclui:
  - Assinatura de mensagens do RabbitMQ para eventos de criação de pedidos.
  - Persistência de faturas no banco de dados PostgreSQL.

- **Kong**: API Gateway para roteamento e gerenciamento de APIs.

- **RabbitMQ**: Broker de mensagens para comunicação assíncrona entre serviços.

- **Jaeger**: Sistema de tracing distribuído para monitoramento.

### Tecnologias Utilizadas

- **Linguagem**: TypeScript
- **Framework HTTP**: Fastify
- **Validação**: Zod
- **Banco de Dados**: PostgreSQL
- **ORM**: Drizzle
- **Mensageria**: RabbitMQ (AMQP)
- **Tracing**: OpenTelemetry, Jaeger
- **Containerização**: Docker, Docker Compose
- **Infraestrutura como Código**: Pulumi (AWS)
- **API Gateway**: Kong

## Estrutura do Projeto

```
node-microservices-platform/
├── app-invoices/          # Serviço de faturas
│   ├── src/
│   │   ├── broker/        # Integração com RabbitMQ
│   │   ├── db/            # Cliente DB e schemas
│   │   └── http/          # Servidor HTTP
│   ├── docker-compose.yml
│   ├── Dockerfile
│   └── package.json
├── app-orders/            # Serviço de pedidos
│   ├── src/
│   │   ├── broker/        # Publicação de mensagens
│   │   ├── db/            # Cliente DB e schemas
│   │   ├── http/          # Servidor HTTP
│   │   └── tracer/        # Configuração de tracing
│   ├── docker-compose.yml
│   ├── Dockerfile
│   └── package.json
├── contracts/             # Contratos compartilhados (mensagens)
├── docker/                # Configurações Docker (Kong)
├── infra/                 # Infraestrutura Pulumi
│   ├── src/
│   │   ├── cluster.ts     # Configuração do cluster ECS
│   │   ├── load-balancer.ts # Load balancer
│   │   ├── images/        # Build de imagens Docker
│   │   └── services/      # Definição de serviços AWS
│   └── Pulumi.yaml
├── docker-compose.yml     # Compose para desenvolvimento local
└── exemplo-com-dominio-e-secret-manager.ts # Exemplo de configuração
```

## Pré-requisitos

- Node.js (>= 14)
- Docker e Docker Compose
- AWS CLI (para infraestrutura)
- Pulumi CLI (>= v3)

## Configuração e Execução Local

1. **Clone o repositório**:
   ```bash
   git clone <url-do-repositorio>
   cd node-microservices-platform
   ```

2. **Instale dependências**:
   ```bash
   cd app-orders && npm install
   cd ../app-invoices && npm install
   cd ../infra && npm install
   ```

3. **Execute os serviços com Docker Compose**:
   ```bash
   docker-compose up --build
   ```

   Isso iniciará:
   - RabbitMQ na porta 5672 (management: 15672)
   - Jaeger UI na porta 16686
   - Kong na porta 8000 (admin: 8001, UI: 8002)

4. **Execute os serviços Node.js**:
   - Para app-orders: `cd app-orders && npm run dev`
   - Para app-invoices: `cd app-invoices && npm run dev`

5. **Teste a API**:
   - Criar pedido: `POST http://localhost:3333/orders` com body `{"amount": 100}`
   - Health check: `GET http://localhost:3333/health`

## Implantação na AWS

1. **Configure AWS credentials**:
   ```bash
   aws configure
   ```

2. **Implante infraestrutura com Pulumi**:
   ```bash
   cd infra
   pulumi stack init dev
   pulumi up
   ```

   Isso provisionará:
   - Cluster ECS
   - Serviços para orders, RabbitMQ e Kong
   - Load balancer

## Modelos de Dados

### Pedidos (Orders)
- `id`: UUID
- `customerId`: UUID (referência ao cliente)
- `amount`: Inteiro
- `status`: Enum (pending, paid, canceled)
- `createdAt`: Timestamp

### Faturas (Invoices)
- `id`: UUID
- `orderId`: UUID (referência ao pedido)
- `createdAt`: Timestamp

### Mensagens
- **OrderCreatedMessage**: Emitido quando um pedido é criado, contendo `orderId`, `amount` e dados do cliente.

## Desenvolvimento

- Use `npm run dev` para desenvolvimento com hot-reload.
- Migrações de banco: Use Drizzle Kit (`drizzle-kit generate` e `drizzle-kit migrate`).
- Tracing: Visualize spans no Jaeger UI (http://localhost:16686).
