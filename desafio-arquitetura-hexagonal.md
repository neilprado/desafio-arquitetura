# Desafio de Arquitetura Hexagonal com Padrões de Projeto

## Sistema de Gerenciamento de Pedidos para E-commerce

---

## 📋 Sumário Executivo

Este documento apresenta um desafio completo de arquitetura de software que combina **Arquitetura Hexagonal (Ports and Adapters)** com diversos **Padrões de Projeto** clássicos. O objetivo é desenvolver um sistema de gerenciamento de pedidos altamente desacoplado, testável e preparado para múltiplas integrações.

**Tecnologias sugeridas:** TypeScript/Node.js, Python, Java ou C#

**Duração estimada:** 40-80 horas (dependendo do nível de complexidade escolhido)

**Abordagem:** Monólito modular evoluindo para microserviços (quando necessário)

---

## 🎯 Contexto do Desafio

Você foi contratado(a) para desenvolver um **sistema de gerenciamento de pedidos** para uma plataforma de e-commerce em crescimento. A empresa precisa de uma arquitetura que permita:

- Integração com múltiplos canais de venda (web, mobile, marketplace)
- Suporte a diferentes gateways de pagamento
- Facilidade para trocar provedores de infraestrutura
- Alta testabilidade e manutenibilidade
- Capacidade de evoluir sem quebrar o sistema existente

---

## 📐 Fundamentos da Arquitetura Hexagonal

### O que é Arquitetura Hexagonal?

A Arquitetura Hexagonal, também conhecida como **Ports and Adapters**, foi proposta por Alistair Cockburn. O conceito central é isolar a lógica de negócio das preocupações técnicas externas.

### Princípios Fundamentais

**Inversão de Dependências:** O domínio não conhece a infraestrutura; a infraestrutura conhece o domínio.

**Portas (Ports):** Interfaces que definem pontos de entrada e saída da aplicação.

**Adaptadores (Adapters):** Implementações concretas que conectam o mundo externo às portas.

**Domínio no Centro:** A lógica de negócio permanece pura, sem dependências de frameworks ou bibliotecas externas.

### Estrutura em Camadas

```
┌─────────────────────────────────────────────────────────┐
│              ADAPTERS (Driving - Entrada)                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │   REST   │  │  GraphQL │  │   CLI    │              │
│  │   API    │  │   API    │  │ Commands │              │
│  └─────┬────┘  └─────┬────┘  └─────┬────┘              │
│        │             │              │                    │
│        └─────────────┼──────────────┘                    │
│                      ▼                                    │
├──────────────────────────────────────────────────────────┤
│                APPLICATION LAYER                         │
│              ┌───────────────┐                           │
│              │   USE CASES   │                           │
│              │     (Ports)   │                           │
│              └───────┬───────┘                           │
│                      ▼                                    │
├──────────────────────────────────────────────────────────┤
│                  DOMAIN LAYER                            │
│              ┌───────────────┐                           │
│              │   ENTITIES    │                           │
│              │   SERVICES    │                           │
│              │  VALUE OBJECTS│                           │
│              │ BUSINESS RULES│                           │
│              └───────┬───────┘                           │
│                      ▼                                    │
├──────────────────────────────────────────────────────────┤
│             ADAPTERS (Driven - Saída)                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │PostgreSQL│  │  MongoDB │  │  Payment │              │
│  │Repository│  │Repository│  │ Gateways │              │
│  └──────────┘  └──────────┘  └──────────┘              │
└─────────────────────────────────────────────────────────┘
```

---

## 🏗️ Arquitetura de Microserviços

### Por que Microserviços?

A arquitetura hexagonal é uma **excelente base para microserviços**. Cada microserviço pode ser construído seguindo os princípios hexagonais, garantindo independência e facilidade de teste.

### Quando Migrar para Microserviços?

**NÃO comece com microserviços!** Inicie com um monólito modular bem estruturado e migre para microserviços quando:

- O time crescer para múltiplos squads
- Necessidade de escalar componentes independentemente
- Diferentes partes do sistema têm ciclos de release distintos
- Necessidade de usar tecnologias diferentes para problemas específicos

### Decomposição de Domínios

Para este desafio, os microserviços sugeridos são:

**Order Service:** Gerenciamento do ciclo de vida de pedidos

**Payment Service:** Processamento de pagamentos e transações

**Catalog Service:** Gestão de produtos e inventário

**Customer Service:** Gerenciamento de clientes e perfis

**Notification Service:** Envio de notificações multicanal

**Shipping Service:** Gestão de entregas e rastreamento

### Arquitetura de Microserviços - Visão Geral

```
┌─────────────────────────────────────────────────────────────┐
│                      API GATEWAY                            │
│          (Kong, AWS API Gateway, nginx)                     │
│   - Autenticação/Autorização                               │
│   - Rate Limiting                                           │
│   - Request Routing                                         │
└────────┬────────────────────────────────┬───────────────────┘
         │                                │
    ┌────▼─────┐                    ┌────▼─────┐
    │ Web App  │                    │ Mobile   │
    └──────────┘                    └──────────┘

┌──────────────────────────────────────────────────────────────┐
│                    SERVICE MESH                              │
│                  (Istio, Linkerd)                            │
└──────────────────────────────────────────────────────────────┘

┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   ORDER     │  │  PAYMENT    │  │  CATALOG    │
│  SERVICE    │  │  SERVICE    │  │  SERVICE    │
│             │  │             │  │             │
│ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │
│ │ Domain  │ │  │ │ Domain  │ │  │ │ Domain  │ │
│ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │
│             │  │             │  │             │
│ PostgreSQL  │  │ PostgreSQL  │  │  MongoDB    │
└──────┬──────┘  └──────┬──────┘  └──────┬──────┘
       │                │                │
       └────────────────┼────────────────┘
                        │
              ┌─────────▼─────────┐
              │  MESSAGE BROKER   │
              │   (RabbitMQ,      │
              │    Kafka, SQS)    │
              └───────────────────┘

┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  CUSTOMER   │  │NOTIFICATION │  │  SHIPPING   │
│  SERVICE    │  │  SERVICE    │  │  SERVICE    │
│             │  │             │  │             │
│ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │
│ │ Domain  │ │  │ │ Domain  │ │  │ │ Domain  │ │
│ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │
│             │  │             │  │             │
│ PostgreSQL  │  │   Redis     │  │ PostgreSQL  │
└─────────────┘  └─────────────┘  └─────────────┘
```

### Comunicação entre Microserviços

**Síncrona (REST/gRPC):**
- Para operações que exigem resposta imediata
- Exemplo: Order Service consulta Catalog Service para validar produtos

**Assíncrona (Mensageria):**
- Para operações que podem ser processadas posteriormente
- Exemplo: Order criado → Payment Service processa pagamento → Notification Service envia email

### Estrutura de um Microserviço com Hexagonal

```
order-service/
├── src/
│   ├── domain/              # Mesmo do monólito
│   │   ├── entities/
│   │   ├── value-objects/
│   │   └── services/
│   │
│   ├── application/         # Mesmo do monólito
│   │   ├── ports/
│   │   └── use-cases/
│   │
│   └── infrastructure/      # Adaptado para microserviços
│       ├── http/
│       │   └── OrderController.ts
│       ├── messaging/
│       │   ├── OrderEventPublisher.ts
│       │   └── PaymentEventConsumer.ts
│       ├── grpc/
│       │   └── OrderServiceImpl.ts
│       └── persistence/
│           └── PostgresOrderRepository.ts
│
├── Dockerfile
├── docker-compose.yml
└── kubernetes/
    ├── deployment.yaml
    └── service.yaml
```

### Padrões para Microserviços

#### 1. API Gateway Pattern

**Propósito:** Ponto único de entrada para clientes, roteia requisições para os microserviços apropriados.

**Responsabilidades:**
- Roteamento de requisições
- Autenticação e autorização
- Rate limiting
- Agregação de respostas
- Cache
- Transformação de protocolos

**Exemplo de Configuração (Kong):**

```yaml
services:
  - name: order-service
    url: http://order-service:3000
    routes:
      - name: orders
        paths:
          - /api/orders
        methods:
          - GET
          - POST
    plugins:
      - name: rate-limiting
        config:
          minute: 100
      - name: jwt
```

#### 2. Service Discovery Pattern

**Propósito:** Permitir que serviços encontrem uns aos outros dinamicamente.

**Opções:**
- **Client-Side Discovery:** Cliente consulta service registry (Consul, Eureka)
- **Server-Side Discovery:** Load balancer consulta registry (Kubernetes Service)

**Exemplo com Consul:**

```typescript
import Consul from 'consul';

class ServiceDiscovery {
  private consul = new Consul();
  
  async registerService(name: string, port: number): Promise<void> {
    await this.consul.agent.service.register({
      id: `${name}-${port}`,
      name: name,
      port: port,
      check: {
        http: `http://localhost:${port}/health`,
        interval: '10s'
      }
    });
  }
  
  async discoverService(name: string): Promise<string> {
    const result = await this.consul.health.service({ service: name, passing: true });
    const service = result[0];
    return `http://${service.Service.Address}:${service.Service.Port}`;
  }
}
```

#### 3. Circuit Breaker Pattern

**Propósito:** Prevenir falhas em cascata quando um serviço está indisponível.

**Estados:**
- **Closed:** Requisições fluem normalmente
- **Open:** Requisições falham imediatamente
- **Half-Open:** Tenta requisições de teste

**Exemplo de Implementação:**

```typescript
class CircuitBreaker {
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  private failures = 0;
  private readonly threshold = 5;
  private readonly timeout = 60000; // 1 minuto
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailTime > this.timeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private onSuccess(): void {
    this.failures = 0;
    this.state = 'CLOSED';
  }
  
  private onFailure(): void {
    this.failures++;
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
      this.lastFailTime = Date.now();
    }
  }
}

// Uso
const paymentCircuit = new CircuitBreaker();

async function processPayment(order: Order): Promise<PaymentResult> {
  return await paymentCircuit.execute(() => 
    paymentService.charge(order.total)
  );
}
```

#### 4. Saga Pattern

**Propósito:** Gerenciar transações distribuídas através de múltiplos serviços.

**Tipos:**
- **Choreography:** Cada serviço publica eventos e reage a eventos de outros
- **Orchestration:** Um coordenador central gerencia o fluxo

**Exemplo - Saga de Criação de Pedido (Choreography):**

```typescript
// Order Service
class CreateOrderSaga {
  async execute(orderData: CreateOrderDTO): Promise<void> {
    // 1. Criar pedido
    const order = await this.orderRepository.save(
      Order.create(orderData)
    );
    
    // 2. Publicar evento
    await this.eventBus.publish(new OrderCreatedEvent(order));
  }
  
  async onPaymentFailed(event: PaymentFailedEvent): Promise<void> {
    // Compensação: cancelar pedido
    const order = await this.orderRepository.findById(event.orderId);
    order.cancel();
    await this.orderRepository.save(order);
  }
}

// Payment Service
class PaymentEventHandler {
  async onOrderCreated(event: OrderCreatedEvent): Promise<void> {
    try {
      // Processar pagamento
      const result = await this.paymentGateway.charge(event.order);
      
      if (result.success) {
        await this.eventBus.publish(new PaymentSuccessEvent(event.orderId));
      } else {
        await this.eventBus.publish(new PaymentFailedEvent(event.orderId));
      }
    } catch (error) {
      await this.eventBus.publish(new PaymentFailedEvent(event.orderId));
    }
  }
}

// Inventory Service
class InventoryEventHandler {
  async onPaymentSuccess(event: PaymentSuccessEvent): Promise<void> {
    try {
      // Reservar estoque
      await this.inventoryService.reserve(event.orderId);
      await this.eventBus.publish(new InventoryReservedEvent(event.orderId));
    } catch (error) {
      // Compensação: estornar pagamento
      await this.eventBus.publish(new InventoryReservationFailedEvent(event.orderId));
    }
  }
}
```

#### 5. CQRS (Command Query Responsibility Segregation)

**Propósito:** Separar operações de leitura e escrita em modelos diferentes.

**Benefícios:**
- Otimizar leitura e escrita independentemente
- Escalar read e write separadamente
- Diferentes modelos de dados para diferentes necessidades

**Exemplo de Implementação:**

```typescript
// Command Side (Write Model)
class CreateOrderCommand {
  constructor(
    public readonly customerId: string,
    public readonly items: OrderItemDTO[]
  ) {}
}

class CreateOrderHandler {
  async handle(command: CreateOrderCommand): Promise<string> {
    const order = Order.create({
      customerId: command.customerId,
      items: command.items
    });
    
    await this.orderRepository.save(order);
    await this.eventBus.publish(new OrderCreatedEvent(order));
    
    return order.id;
  }
}

// Query Side (Read Model)
interface OrderReadModel {
  id: string;
  customerName: string;
  totalAmount: number;
  status: string;
  items: Array<{
    productName: string;
    quantity: number;
    price: number;
  }>;
}

class OrderQueryService {
  async getOrderById(orderId: string): Promise<OrderReadModel> {
    // Consulta banco de leitura otimizado (view materializada)
    return await this.readDatabase.query(
      'SELECT * FROM order_view WHERE id = $1',
      [orderId]
    );
  }
  
  async listOrdersByCustomer(customerId: string): Promise<OrderReadModel[]> {
    return await this.readDatabase.query(
      'SELECT * FROM order_view WHERE customer_id = $1 ORDER BY created_at DESC',
      [customerId]
    );
  }
}

// Event Handler atualiza read model
class OrderReadModelUpdater {
  async onOrderCreated(event: OrderCreatedEvent): Promise<void> {
    await this.readDatabase.execute(`
      INSERT INTO order_view (id, customer_name, total_amount, status)
      VALUES ($1, $2, $3, $4)
    `, [event.order.id, event.customerName, event.order.total, 'CREATED']);
  }
}
```

#### 6. Event Sourcing

**Propósito:** Armazenar o histórico de mudanças como sequência de eventos ao invés do estado atual.

**Benefícios:**
- Auditoria completa
- Capacidade de reconstruir estado passado
- Event replay para debugging
- Projeções múltiplas do mesmo evento

**Exemplo de Implementação:**

```typescript
// Events
abstract class DomainEvent {
  constructor(
    public readonly aggregateId: string,
    public readonly occurredOn: Date = new Date()
  ) {}
}

class OrderCreatedEvent extends DomainEvent {
  constructor(
    aggregateId: string,
    public readonly customerId: string,
    public readonly items: OrderItem[]
  ) {
    super(aggregateId);
  }
}

class OrderPaidEvent extends DomainEvent {
  constructor(
    aggregateId: string,
    public readonly paymentId: string,
    public readonly amount: number
  ) {
    super(aggregateId);
  }
}

// Event Store
interface EventStore {
  append(streamId: string, events: DomainEvent[]): Promise<void>;
  getEvents(streamId: string): Promise<DomainEvent[]>;
}

class PostgresEventStore implements EventStore {
  async append(streamId: string, events: DomainEvent[]): Promise<void> {
    for (const event of events) {
      await this.db.execute(`
        INSERT INTO events (stream_id, event_type, event_data, occurred_on)
        VALUES ($1, $2, $3, $4)
      `, [streamId, event.constructor.name, JSON.stringify(event), event.occurredOn]);
    }
  }
  
  async getEvents(streamId: string): Promise<DomainEvent[]> {
    const rows = await this.db.query(
      'SELECT * FROM events WHERE stream_id = $1 ORDER BY occurred_on',
      [streamId]
    );
    
    return rows.map(row => this.deserializeEvent(row));
  }
}

// Aggregate reconstruído a partir de eventos
class Order {
  private events: DomainEvent[] = [];
  
  static rehydrate(events: DomainEvent[]): Order {
    const order = new Order();
    events.forEach(event => order.apply(event));
    return order;
  }
  
  private apply(event: DomainEvent): void {
    if (event instanceof OrderCreatedEvent) {
      this.id = event.aggregateId;
      this.customerId = event.customerId;
      this.items = event.items;
      this.status = OrderStatus.CREATED;
    } else if (event instanceof OrderPaidEvent) {
      this.status = OrderStatus.PAID;
      this.paymentId = event.paymentId;
    }
    // ... outros eventos
  }
  
  pay(paymentId: string): void {
    const event = new OrderPaidEvent(this.id, paymentId, this.total);
    this.apply(event);
    this.events.push(event);
  }
  
  getUncommittedEvents(): DomainEvent[] {
    return this.events;
  }
}
```

#### 7. Backend for Frontend (BFF)

**Propósito:** Criar APIs específicas para cada tipo de cliente (web, mobile, desktop).

**Benefícios:**
- Otimizar payloads para cada plataforma
- Lógica de agregação específica por cliente
- Evolução independente de cada interface

**Estrutura:**

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Web App    │     │ Mobile App  │     │Desktop App  │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                    │
┌──────▼──────┐     ┌──────▼──────┐     ┌──────▼──────┐
│   Web BFF   │     │ Mobile BFF  │     │Desktop BFF  │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                    │
       └───────────────────┼────────────────────┘
                           │
              ┌────────────▼────────────┐
              │   Core Services         │
              │ (Order, Payment, etc)   │
              └─────────────────────────┘
```

**Exemplo:**

```typescript
// Mobile BFF - dados otimizados para mobile
class MobileOrderBFF {
  async getOrderDetails(orderId: string): Promise<MobileOrderView> {
    // Agrega dados de múltiplos serviços
    const [order, customer, tracking] = await Promise.all([
      this.orderService.getOrder(orderId),
      this.customerService.getCustomer(order.customerId),
      this.shippingService.getTracking(orderId)
    ]);
    
    // Retorna apenas dados essenciais para mobile
    return {
      id: order.id,
      status: order.status,
      total: order.total,
      customerName: customer.name,
      deliveryEstimate: tracking.estimatedDate,
      // Sem detalhes completos de produtos
      itemCount: order.items.length
    };
  }
}

// Web BFF - dados completos para desktop
class WebOrderBFF {
  async getOrderDetails(orderId: string): Promise<WebOrderView> {
    const [order, customer, products, tracking, invoices] = await Promise.all([
      this.orderService.getOrder(orderId),
      this.customerService.getCustomer(order.customerId),
      this.catalogService.getProducts(order.items.map(i => i.productId)),
      this.shippingService.getTracking(orderId),
      this.billingService.getInvoices(orderId)
    ]);
    
    // Retorna dados completos e ricos
    return {
      order: order,
      customer: {
        name: customer.name,
        email: customer.email,
        phone: customer.phone,
        address: customer.address
      },
      items: order.items.map(item => ({
        ...item,
        product: products.find(p => p.id === item.productId)
      })),
      tracking: tracking,
      invoices: invoices
    };
  }
}
```

### Observabilidade em Microserviços

#### Distributed Tracing

**Propósito:** Rastrear requisições através de múltiplos serviços.

**Ferramentas:** Jaeger, Zipkin, AWS X-Ray

**Exemplo com OpenTelemetry:**

```typescript
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('order-service');

class CreateOrderHandler {
  async handle(command: CreateOrderCommand): Promise<string> {
    const span = tracer.startSpan('create-order');
    
    try {
      span.setAttribute('customer.id', command.customerId);
      span.setAttribute('items.count', command.items.length);
      
      const order = await this.createOrder(command);
      span.addEvent('order-created', { orderId: order.id });
      
      await this.publishEvent(order);
      span.addEvent('event-published');
      
      return order.id;
    } catch (error) {
      span.recordException(error);
      throw error;
    } finally {
      span.end();
    }
  }
}
```

#### Métricas

**Propósito:** Monitorar saúde e performance dos serviços.

**Métricas Importantes:**
- **RED:** Rate, Errors, Duration
- **USE:** Utilization, Saturation, Errors
- **Métricas de negócio:** Pedidos/hora, valor médio, taxa de conversão

**Exemplo com Prometheus:**

```typescript
import { Counter, Histogram, register } from 'prom-client';

const orderCounter = new Counter({
  name: 'orders_created_total',
  help: 'Total de pedidos criados',
  labelNames: ['status']
});

const orderDuration = new Histogram({
  name: 'order_creation_duration_seconds',
  help: 'Duração da criação de pedidos',
  buckets: [0.1, 0.5, 1, 2, 5]
});

class CreateOrderHandler {
  async handle(command: CreateOrderCommand): Promise<string> {
    const end = orderDuration.startTimer();
    
    try {
      const order = await this.createOrder(command);
      orderCounter.inc({ status: 'success' });
      return order.id;
    } catch (error) {
      orderCounter.inc({ status: 'error' });
      throw error;
    } finally {
      end();
    }
  }
}

// Endpoint de métricas
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});
```

#### Logging Estruturado

**Propósito:** Facilitar busca e análise de logs em ambiente distribuído.

**Exemplo:**

```typescript
import winston from 'winston';

const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'application.log' })
  ]
});

class CreateOrderHandler {
  async handle(command: CreateOrderCommand): Promise<string> {
    const correlationId = generateCorrelationId();
    
    logger.info('Creating order', {
      correlationId,
      customerId: command.customerId,
      itemCount: command.items.length
    });
    
    try {
      const order = await this.createOrder(command);
      
      logger.info('Order created successfully', {
        correlationId,
        orderId: order.id,
        totalAmount: order.total
      });
      
      return order.id;
    } catch (error) {
      logger.error('Failed to create order', {
        correlationId,
        error: error.message,
        stack: error.stack
      });
      throw error;
    }
  }
}
```

### Deploy e Orquestração

#### Docker Compose (Desenvolvimento Local)

**Arquivo completo: `docker-compose.yml`**

```yaml
version: '3.8'

services:
  # ============================================================================
  # API Gateway
  # ============================================================================
  api-gateway:
    image: kong:3.4
    ports:
      - "8000:8000"   # Proxy
      - "8001:8001"   # Admin API
      - "8443:8443"   # Proxy SSL
      - "8444:8444"   # Admin API SSL
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: postgres
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kong
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    depends_on:
      - postgres
    networks:
      - ecommerce-network

  # ============================================================================
  # Microserviços
  # ============================================================================
  order-service:
    build:
      context: ./order-service
      dockerfile: Dockerfile
    ports:
      - "3001:3000"
    environment:
      NODE_ENV: development
      PORT: 3000
      DATABASE_URL: postgresql://postgres:password@postgres:5432/orders
      RABBITMQ_URL: amqp://guest:guest@rabbitmq:5672
      REDIS_URL: redis://redis:6379
      PAYMENT_SERVICE_URL: http://payment-service:3000
      CATALOG_SERVICE_URL: http://catalog-service:3000
      JAEGER_AGENT_HOST: jaeger
      JAEGER_AGENT_PORT: 6831
    depends_on:
      postgres:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./order-service:/app
      - /app/node_modules
    networks:
      - ecommerce-network
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s

  payment-service:
    build:
      context: ./payment-service
      dockerfile: Dockerfile
    ports:
      - "3002:3000"
    environment:
      NODE_ENV: development
      PORT: 3000
      DATABASE_URL: postgresql://postgres:password@postgres:5432/payments
      RABBITMQ_URL: amqp://guest:guest@rabbitmq:5672
      REDIS_URL: redis://redis:6379
      STRIPE_SECRET_KEY: ${STRIPE_SECRET_KEY:-sk_test_dummy}
      PAYPAL_CLIENT_ID: ${PAYPAL_CLIENT_ID:-dummy}
      PAYPAL_SECRET: ${PAYPAL_SECRET:-dummy}
      JAEGER_AGENT_HOST: jaeger
      JAEGER_AGENT_PORT: 6831
    depends_on:
      postgres:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./payment-service:/app
      - /app/node_modules
    networks:
      - ecommerce-network
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3

  catalog-service:
    build:
      context: ./catalog-service
      dockerfile: Dockerfile
    ports:
      - "3003:3000"
    environment:
      NODE_ENV: development
      PORT: 3000
      MONGODB_URL: mongodb://mongo:27017/catalog
      RABBITMQ_URL: amqp://guest:guest@rabbitmq:5672
      REDIS_URL: redis://redis:6379
      JAEGER_AGENT_HOST: jaeger
      JAEGER_AGENT_PORT: 6831
    depends_on:
      mongo:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./catalog-service:/app
      - /app/node_modules
    networks:
      - ecommerce-network
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3

  customer-service:
    build:
      context: ./customer-service
      dockerfile: Dockerfile
    ports:
      - "3004:3000"
    environment:
      NODE_ENV: development
      PORT: 3000
      DATABASE_URL: postgresql://postgres:password@postgres:5432/customers
      RABBITMQ_URL: amqp://guest:guest@rabbitmq:5672
      REDIS_URL: redis://redis:6379
      JAEGER_AGENT_HOST: jaeger
      JAEGER_AGENT_PORT: 6831
    depends_on:
      postgres:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./customer-service:/app
      - /app/node_modules
    networks:
      - ecommerce-network
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3

  notification-service:
    build:
      context: ./notification-service
      dockerfile: Dockerfile
    ports:
      - "3005:3000"
    environment:
      NODE_ENV: development
      PORT: 3000
      RABBITMQ_URL: amqp://guest:guest@rabbitmq:5672
      REDIS_URL: redis://redis:6379
      SMTP_HOST: mailhog
      SMTP_PORT: 1025
      TWILIO_ACCOUNT_SID: ${TWILIO_ACCOUNT_SID:-dummy}
      TWILIO_AUTH_TOKEN: ${TWILIO_AUTH_TOKEN:-dummy}
      JAEGER_AGENT_HOST: jaeger
      JAEGER_AGENT_PORT: 6831
    depends_on:
      rabbitmq:
        condition: service_healthy
      redis:
        condition: service_started
      mailhog:
        condition: service_started
    volumes:
      - ./notification-service:/app
      - /app/node_modules
    networks:
      - ecommerce-network
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3

  shipping-service:
    build:
      context: ./shipping-service
      dockerfile: Dockerfile
    ports:
      - "3006:3000"
    environment:
      NODE_ENV: development
      PORT: 3000
      DATABASE_URL: postgresql://postgres:password@postgres:5432/shipping
      RABBITMQ_URL: amqp://guest:guest@rabbitmq:5672
      REDIS_URL: redis://redis:6379
      JAEGER_AGENT_HOST: jaeger
      JAEGER_AGENT_PORT: 6831
    depends_on:
      postgres:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./shipping-service:/app
      - /app/node_modules
    networks:
      - ecommerce-network
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s

  # ============================================================================
  # Bancos de Dados
  # ============================================================================
  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_MULTIPLE_DATABASES: orders,payments,customers,shipping,kong
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./scripts/init-databases.sh:/docker-entrypoint-initdb.d/init-databases.sh
    networks:
      - ecommerce-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
      MONGO_INITDB_DATABASE: catalog
    volumes:
      - mongo-data:/data/db
    networks:
      - ecommerce-network
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - ecommerce-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ============================================================================
  # Message Broker
  # ============================================================================
  rabbitmq:
    image: rabbitmq:3.12-management-alpine
    ports:
      - "5672:5672"   # AMQP
      - "15672:15672" # Management UI
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
      - ./config/rabbitmq/definitions.json:/etc/rabbitmq/definitions.json
      - ./config/rabbitmq/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf
    networks:
      - ecommerce-network
    healthcheck:
      test: rabbitmq-diagnostics -q ping
      interval: 10s
      timeout: 5s
      retries: 5

  # ============================================================================
  # Observabilidade
  # ============================================================================
  jaeger:
    image: jaegertracing/all-in-one:1.51
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686"     # UI
      - "14268:14268"
      - "14250:14250"
      - "9411:9411"
    environment:
      COLLECTOR_ZIPKIN_HOST_PORT: :9411
    networks:
      - ecommerce-network

  prometheus:
    image: prom/prometheus:v2.48.0
    ports:
      - "9090:9090"
    volumes:
      - ./config/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    networks:
      - ecommerce-network

  grafana:
    image: grafana/grafana:10.2.2
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_USER: admin
      GF_SECURITY_ADMIN_PASSWORD: admin
      GF_INSTALL_PLUGINS: grafana-piechart-panel
    volumes:
      - grafana-data:/var/lib/grafana
      - ./config/grafana/provisioning:/etc/grafana/provisioning
      - ./config/grafana/dashboards:/var/lib/grafana/dashboards
    depends_on:
      - prometheus
    networks:
      - ecommerce-network

  # ============================================================================
  # Ferramentas de Desenvolvimento
  # ============================================================================
  mailhog:
    image: mailhog/mailhog:v1.0.1
    ports:
      - "1025:1025" # SMTP
      - "8025:8025" # Web UI
    networks:
      - ecommerce-network

  adminer:
    image: adminer:4.8.1
    ports:
      - "8080:8080"
    environment:
      ADMINER_DEFAULT_SERVER: postgres
    networks:
      - ecommerce-network

  mongo-express:
    image: mongo-express:1.0.2
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: password
      ME_CONFIG_MONGODB_URL: mongodb://admin:password@mongo:27017/
    depends_on:
      - mongo
    networks:
      - ecommerce-network

volumes:
  postgres-data:
  mongo-data:
  redis-data:
  rabbitmq-data:
  prometheus-data:
  grafana-data:

networks:
  ecommerce-network:
    driver: bridge
```

### Scripts e Configurações de Apoio

#### scripts/init-databases.sh

```bash
#!/bin/bash
set -e

# Script para criar múltiplos bancos no PostgreSQL

psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" <<-EOSQL
    CREATE DATABASE orders;
    CREATE DATABASE payments;
    CREATE DATABASE customers;
    CREATE DATABASE shipping;
    CREATE DATABASE kong;
    
    -- Criar usuário kong para o API Gateway
    CREATE USER kong WITH PASSWORD 'kong';
    GRANT ALL PRIVILEGES ON DATABASE kong TO kong;
EOSQL

echo "✅ Databases criados com sucesso!"
```

#### config/rabbitmq/definitions.json

```json
{
  "rabbit_version": "3.12.0",
  "users": [
    {
      "name": "guest",
      "password_hash": "guest",
      "tags": "administrator"
    }
  ],
  "vhosts": [
    {
      "name": "/"
    }
  ],
  "exchanges": [
    {
      "name": "orders",
      "vhost": "/",
      "type": "topic",
      "durable": true,
      "auto_delete": false,
      "internal": false
    },
    {
      "name": "payments",
      "vhost": "/",
      "type": "topic",
      "durable": true,
      "auto_delete": false,
      "internal": false
    },
    {
      "name": "notifications",
      "vhost": "/",
      "type": "fanout",
      "durable": true,
      "auto_delete": false,
      "internal": false
    }
  ],
  "queues": [
    {
      "name": "order.created",
      "vhost": "/",
      "durable": true,
      "auto_delete": false
    },
    {
      "name": "payment.process",
      "vhost": "/",
      "durable": true,
      "auto_delete": false
    },
    {
      "name": "notification.email",
      "vhost": "/",
      "durable": true,
      "auto_delete": false
    },
    {
      "name": "notification.sms",
      "vhost": "/",
      "durable": true,
      "auto_delete": false
    }
  ],
  "bindings": [
    {
      "source": "orders",
      "vhost": "/",
      "destination": "order.created",
      "destination_type": "queue",
      "routing_key": "order.created"
    },
    {
      "source": "orders",
      "vhost": "/",
      "destination": "payment.process",
      "destination_type": "queue",
      "routing_key": "order.created"
    },
    {
      "source": "payments",
      "vhost": "/",
      "destination": "notification.email",
      "destination_type": "queue",
      "routing_key": "payment.success"
    }
  ]
}
```

#### config/rabbitmq/rabbitmq.conf

```conf
# RabbitMQ Configuration for Development

# Importar definições na inicialização
management.load_definitions = /etc/rabbitmq/definitions.json

# Configurações de memória
vm_memory_high_watermark.relative = 0.8

# Logs
log.file.level = info
log.console = true
log.console.level = info
```

#### config/prometheus/prometheus.yml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'order-service'
    static_configs:
      - targets: ['order-service:3000']
    metrics_path: '/metrics'

  - job_name: 'payment-service'
    static_configs:
      - targets: ['payment-service:3000']
    metrics_path: '/metrics'

  - job_name: 'catalog-service'
    static_configs:
      - targets: ['catalog-service:3000']
    metrics_path: '/metrics'

  - job_name: 'customer-service'
    static_configs:
      - targets: ['customer-service:3000']
    metrics_path: '/metrics'

  - job_name: 'notification-service'
    static_configs:
      - targets: ['notification-service:3000']
    metrics_path: '/metrics'

  - job_name: 'shipping-service'
    static_configs:
      - targets: ['shipping-service:3000']
    metrics_path: '/metrics'

  - job_name: 'rabbitmq'
    static_configs:
      - targets: ['rabbitmq:15692']
```

### Dockerfile Exemplo (Node.js/TypeScript)

**order-service/Dockerfile**

```dockerfile
# ============================================================================
# Estágio de Build
# ============================================================================
FROM node:20-alpine AS builder

WORKDIR /app

# Copiar arquivos de dependências
COPY package*.json ./
COPY tsconfig.json ./

# Instalar dependências
RUN npm ci --only=production && \
    npm cache clean --force

# Copiar código fonte
COPY src ./src

# Build TypeScript
RUN npm run build

# ============================================================================
# Imagem de Produção
# ============================================================================
FROM node:20-alpine AS production

WORKDIR /app

# Instalar apenas dependências de runtime
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force

# Copiar build do estágio anterior
COPY --from=builder /app/dist ./dist

# Criar usuário não-root para segurança
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 && \
    chown -R nodejs:nodejs /app

USER nodejs

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

CMD ["node", "dist/main.js"]

# ============================================================================
# Imagem de Desenvolvimento
# ============================================================================
FROM node:20-alpine AS development

WORKDIR /app

# Instalar dependências de desenvolvimento
COPY package*.json ./
RUN npm install

# Copiar código fonte
COPY . .

EXPOSE 3000

# Hot reload com nodemon ou ts-node-dev
CMD ["npm", "run", "dev"]
```

### Makefile para Facilitar Comandos

```makefile
.PHONY: help up down restart logs ps build clean migrate test

help: ## 📚 Mostra esta ajuda
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

up: ## 🚀 Inicia todos os serviços
	@echo "🚀 Iniciando ambiente de desenvolvimento..."
	docker-compose up -d
	@echo "✅ Ambiente iniciado! Execute 'make urls' para ver os endpoints"

down: ## 🛑 Para todos os serviços
	@echo "🛑 Parando todos os serviços..."
	docker-compose down

restart: ## 🔄 Reinicia todos os serviços
	@echo "🔄 Reiniciando serviços..."
	docker-compose restart

logs: ## 📋 Mostra logs de todos os serviços
	docker-compose logs -f

logs-order: ## 📋 Logs do order-service
	docker-compose logs -f order-service

logs-payment: ## 📋 Logs do payment-service
	docker-compose logs -f payment-service

logs-catalog: ## 📋 Logs do catalog-service
	docker-compose logs -f catalog-service

ps: ## 📊 Lista todos os containers
	docker-compose ps

build: ## 🔨 Builda todas as imagens
	@echo "🔨 Buildando imagens Docker..."
	docker-compose build

build-nocache: ## 🔨 Build sem cache
	@echo "🔨 Buildando sem cache..."
	docker-compose build --no-cache

clean: ## 🧹 Remove tudo (containers, volumes, imagens)
	@echo "🧹 Limpando ambiente..."
	docker-compose down -v --rmi all
	@echo "✅ Ambiente limpo!"

prune: ## 🗑️  Remove recursos Docker não utilizados
	@echo "🗑️  Removendo recursos não utilizados..."
	docker system prune -af --volumes

migrate: ## 🗄️  Executa migrations em todos os serviços
	@echo "🗄️  Executando migrations..."
	docker-compose exec order-service npm run migrate
	docker-compose exec payment-service npm run migrate
	docker-compose exec customer-service npm run migrate
	docker-compose exec shipping-service npm run migrate
	@echo "✅ Migrations concluídas!"

seed: ## 🌱 Popula bancos com dados de teste
	@echo "🌱 Populando bancos de dados..."
	docker-compose exec order-service npm run seed
	docker-compose exec catalog-service npm run seed
	docker-compose exec customer-service npm run seed
	@echo "✅ Dados de teste inseridos!"

test: ## 🧪 Executa testes unitários
	@echo "🧪 Executando testes..."
	docker-compose exec order-service npm test
	docker-compose exec payment-service npm test
	docker-compose exec catalog-service npm test

test-e2e: ## 🧪 Executa testes E2E
	@echo "🧪 Executando testes E2E..."
	docker-compose exec order-service npm run test:e2e

test-integration: ## 🧪 Testes de integração
	@echo "🧪 Executando testes de integração..."
	npm run test:integration

shell-order: ## 🐚 Shell no order-service
	docker-compose exec order-service sh

shell-payment: ## 🐚 Shell no payment-service
	docker-compose exec payment-service sh

shell-catalog: ## 🐚 Shell no catalog-service
	docker-compose exec catalog-service sh

db-psql: ## 🗄️  Conecta ao PostgreSQL
	docker-compose exec postgres psql -U postgres

db-mongo: ## 🗄️  Conecta ao MongoDB
	docker-compose exec mongo mongosh -u admin -p password

db-redis: ## 🗄️  Conecta ao Redis CLI
	docker-compose exec redis redis-cli

rabbitmq-reset: ## 🐰 Reseta o RabbitMQ
	@echo "🐰 Resetando RabbitMQ..."
	docker-compose restart rabbitmq
	@echo "✅ RabbitMQ resetado!"

health: ## 💚 Verifica saúde de todos os serviços
	@echo "💚 Verificando saúde dos serviços..."
	@echo "\n📦 Order Service:"
	@curl -s http://localhost:3001/health | jq . || echo "❌ Indisponível"
	@echo "\n💳 Payment Service:"
	@curl -s http://localhost:3002/health | jq . || echo "❌ Indisponível"
	@echo "\n📚 Catalog Service:"
	@curl -s http://localhost:3003/health | jq . || echo "❌ Indisponível"
	@echo "\n👤 Customer Service:"
	@curl -s http://localhost:3004/health | jq . || echo "❌ Indisponível"
	@echo "\n📧 Notification Service:"
	@curl -s http://localhost:3005/health | jq . || echo "❌ Indisponível"
	@echo "\n🚚 Shipping Service:"
	@curl -s http://localhost:3006/health | jq . || echo "❌ Indisponível"

urls: ## 🌐 Mostra URLs de todos os serviços
	@echo ""
	@echo "╔════════════════════════════════════════════════════════╗"
	@echo "║           🎯 URLs dos Serviços                         ║"
	@echo "╚════════════════════════════════════════════════════════╝"
	@echo ""
	@echo "📦 Microserviços:"
	@echo "  ├─ Order Service:        http://localhost:3001"
	@echo "  ├─ Payment Service:      http://localhost:3002"
	@echo "  ├─ Catalog Service:      http://localhost:3003"
	@echo "  ├─ Customer Service:     http://localhost:3004"
	@echo "  ├─ Notification Service: http://localhost:3005"
	@echo "  └─ Shipping Service:     http://localhost:3006"
	@echo ""
	@echo "🎛️  Infraestrutura:"
	@echo "  ├─ API Gateway (Kong):   http://localhost:8000"
	@echo "  ├─ PostgreSQL (Adminer): http://localhost:8080"
	@echo "  └─ MongoDB (Express):    http://localhost:8081"
	@echo ""
	@echo "📊 Observabilidade:"
	@echo "  ├─ Jaeger UI:            http://localhost:16686"
	@echo "  ├─ Grafana:              http://localhost:3000 (admin/admin)"
	@echo "  ├─ Prometheus:           http://localhost:9090"
	@echo "  └─ RabbitMQ Management:  http://localhost:15672 (guest/guest)"
	@echo ""
	@echo "🛠️  Ferramentas Dev:"
	@echo "  └─ MailHog:              http://localhost:8025"
	@echo ""

install: ## 📥 Instala dependências de todos os serviços
	@echo "📥 Instalando dependências..."
	cd order-service && npm install
	cd payment-service && npm install
	cd catalog-service && npm install
	cd customer-service && npm install
	cd notification-service && npm install
	cd shipping-service && npm install
	@echo "✅ Dependências instaladas!"

format: ## 💅 Formata código de todos os serviços
	@echo "💅 Formatando código..."
	cd order-service && npm run format
	cd payment-service && npm run format
	cd catalog-service && npm run format
	@echo "✅ Código formatado!"

lint: ## 🔍 Executa linter
	@echo "🔍 Executando linter..."
	cd order-service && npm run lint
	cd payment-service && npm run lint
	cd catalog-service && npm run lint

setup: ## 🎬 Setup completo do projeto
	@echo "🎬 Iniciando setup completo..."
	@make install
	@chmod +x scripts/*.sh
	@make build
	@make up
	@sleep 10
	@make migrate
	@make seed
	@make urls
	@echo "✅ Setup completo! Ambiente pronto para desenvolvimento."

reset: ## 🔄 Reset completo (limpa e recria tudo)
	@echo "🔄 Resetando ambiente..."
	@make clean
	@make setup
	@echo "✅ Ambiente resetado com sucesso!"
```

### Guia de Uso Passo a Passo

#### 1. 📦 Setup Inicial

```bash
# Clone o repositório
git clone <seu-repo>
cd ecommerce-microservices

# Setup completo automatizado
make setup

# Ou manualmente:
chmod +x scripts/*.sh
make build
make up
make migrate
make seed
```

#### 2. 🚀 Iniciando o Ambiente

```bash
# Iniciar todos os serviços
make up

# Verificar status
make ps

# Ver logs em tempo real
make logs

# Ver URLs de acesso
make urls
```

#### 3. 🔍 Verificando a Saúde

```bash
# Verificar todos os serviços
make health

# Verificar serviço específico
curl http://localhost:3001/health | jq .
```

#### 4. 💻 Desenvolvimento

```bash
# Ver logs de um serviço específico
make logs-order

# Entrar no shell de um container
make shell-order

# Executar testes
make test

# Testes E2E
make test-e2e

# Formatar código
make format

# Lint
make lint
```

#### 5. 🗄️ Trabalhando com Bancos

```bash
# Conectar ao PostgreSQL
make db-psql

# Dentro do psql:
\l                          # Listar databases
\c orders                   # Conectar ao database orders
\dt                         # Listar tabelas
SELECT * FROM orders;       # Query

# Conectar ao MongoDB
make db-mongo

# Conectar ao Redis
make db-redis
```

#### 6. 🐰 RabbitMQ

```bash
# Acessar UI: http://localhost:15672 (guest/guest)

# Resetar RabbitMQ
make rabbitmq-reset

# Ver logs do RabbitMQ
docker-compose logs -f rabbitmq
```

#### 7. 📊 Observabilidade

```bash
# Jaeger (Distributed Tracing)
# Acesse: http://localhost:16686

# Grafana (Dashboards)
# Acesse: http://localhost:3000 (admin/admin)

# Prometheus (Métricas)
# Acesse: http://localhost:9090
```

#### 8. 📧 Testando Notificações

```bash
# MailHog captura todos os emails
# Acesse: http://localhost:8025

# Enviar email de teste
curl -X POST http://localhost:3005/test-email
```

#### 9. 🔄 Hot Reload

Os serviços estão configurados com hot reload. Qualquer mudança no código será refletida automaticamente:

```bash
# Edite um arquivo
vim order-service/src/controllers/OrderController.ts

# O serviço recarrega automaticamente
# Acompanhe os logs
make logs-order
```

#### 10. 🧹 Limpeza

```bash
# Parar serviços
make down

# Limpar tudo (containers, volumes, imagens)
make clean

# Limpar recursos Docker não utilizados
make prune

# Reset completo
make reset
```

### Configurando Variáveis de Ambiente

Crie um arquivo `.env` na raiz do projeto:

```bash
# .env

# Payment Gateways
STRIPE_SECRET_KEY=sk_test_seu_token_aqui
PAYPAL_CLIENT_ID=seu_client_id
PAYPAL_SECRET=seu_secret

# SMS/Notifications
TWILIO_ACCOUNT_SID=seu_account_sid
TWILIO_AUTH_TOKEN=seu_auth_token

# Ambiente
NODE_ENV=development
LOG_LEVEL=debug
```

### Testando Fluxo Completo

```bash
# 1. Criar um pedido
curl -X POST http://localhost:3001/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "customer-123",
    "items": [
      {
        "productId": "product-456",
        "quantity": 2,
        "price": 99.90
      }
    ]
  }'

# 2. Ver no Jaeger a trace completa
# Acesse: http://localhost:16686
# Selecione "order-service" e busque traces recentes

# 3. Ver logs estruturados
make logs-order
make logs-payment

# 4. Ver email no MailHog
# Acesse: http://localhost:8025

# 5. Ver métricas no Grafana
# Acesse: http://localhost:3000
```

### Troubleshooting

#### Serviço não inicia

```bash
# Ver logs detalhados
docker-compose logs order-service

# Verificar saúde do banco
docker-compose exec postgres pg_isready

# Recriar container
docker-compose up -d --force-recreate order-service
```

#### Problemas de rede

```bash
# Inspecionar rede
docker network inspect ecommerce-microservices_ecommerce-network

# Testar comunicação entre serviços
docker-compose exec order-service ping payment-service
docker-compose exec order-service wget -O- http://payment-service:3000/health
```

#### RabbitMQ não conecta

```bash
# Verificar se está rodando
docker-compose ps rabbitmq

# Resetar
make rabbitmq-reset

# Ver configuração
docker-compose exec rabbitmq cat /etc/rabbitmq/rabbitmq.conf
```

#### Porta em uso

```bash
# Verificar o que está usando a porta
lsof -i :3001

# Matar processo
kill -9 <PID>

# Ou mudar porta no docker-compose.yml
```

---

#### Kubernetes (Produção)

```yaml
# order-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: order-service:1.0.0
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: order-db-secret
              key: url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: ClusterIP

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## 🎨 Padrões de Projeto a Implementar

### 1. Strategy Pattern (Estratégia)

**Propósito:** Definir uma família de algoritmos, encapsular cada um deles e torná-los intercambiáveis.

**Aplicação no Desafio:**

Implementar diferentes estratégias de cálculo de desconto:

- Desconto percentual (ex: 10% de desconto)
- Desconto fixo (ex: R$ 50,00 de desconto)
- Desconto progressivo (quanto maior o valor, maior o desconto)
- Desconto por categoria de cliente (VIP, regular, novo)

**Exemplo de Estrutura:**

```typescript
interface DiscountStrategy {
  calculate(order: Order): Money;
}

class PercentageDiscount implements DiscountStrategy {
  constructor(private percentage: number) {}
  
  calculate(order: Order): Money {
    return order.subtotal.multiply(this.percentage / 100);
  }
}

class FixedDiscount implements DiscountStrategy {
  constructor(private amount: Money) {}
  
  calculate(order: Order): Money {
    return this.amount;
  }
}
```

### 2. Factory Pattern (Fábrica)

**Propósito:** Fornecer uma interface para criar objetos em uma superclasse, mas permitir que subclasses alterem o tipo de objetos criados.

**Aplicação no Desafio:**

- Factory para criar diferentes gateways de pagamento (Stripe, PayPal, PagSeguro)
- Factory para criar diferentes tipos de notificações (Email, SMS, Push)

**Exemplo de Estrutura:**

```typescript
interface PaymentGateway {
  processPayment(order: Order, paymentMethod: PaymentMethod): Promise<PaymentResult>;
}

class PaymentGatewayFactory {
  create(provider: string): PaymentGateway {
    switch(provider) {
      case 'stripe': return new StripeGateway();
      case 'paypal': return new PayPalGateway();
      case 'pagseguro': return new PagSeguroGateway();
      default: throw new Error(`Unknown provider: ${provider}`);
    }
  }
}
```

### 3. Repository Pattern (Repositório)

**Propósito:** Encapsular a lógica de acesso a dados e fornecer uma interface de coleção para objetos de domínio.

**Aplicação no Desafio:**

- Abstrair persistência de pedidos
- Permitir troca entre PostgreSQL e MongoDB sem alterar o domínio
- Facilitar testes com implementações in-memory

**Exemplo de Estrutura:**

```typescript
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order | null>;
  findByCustomer(customerId: string): Promise<Order[]>;
  delete(id: string): Promise<void>;
}

class PostgresOrderRepository implements OrderRepository {
  // Implementação específica do PostgreSQL
}

class MongoOrderRepository implements OrderRepository {
  // Implementação específica do MongoDB
}

class InMemoryOrderRepository implements OrderRepository {
  // Implementação para testes
}
```

### 4. Observer Pattern (Observador)

**Propósito:** Definir uma dependência um-para-muitos entre objetos para que quando um objeto muda de estado, todos os seus dependentes sejam notificados.

**Aplicação no Desafio:**

- Sistema de eventos para mudanças de status de pedido
- Notificar múltiplos serviços quando um pedido é criado, pago, enviado ou cancelado

**Exemplo de Estrutura:**

```typescript
interface OrderEventObserver {
  onOrderCreated(order: Order): void;
  onOrderPaid(order: Order): void;
  onOrderShipped(order: Order): void;
  onOrderCancelled(order: Order): void;
}

class OrderEventPublisher {
  private observers: OrderEventObserver[] = [];
  
  subscribe(observer: OrderEventObserver): void {
    this.observers.push(observer);
  }
  
  notifyOrderCreated(order: Order): void {
    this.observers.forEach(obs => obs.onOrderCreated(order));
  }
}

class EmailNotificationObserver implements OrderEventObserver {
  onOrderCreated(order: Order): void {
    // Enviar email de confirmação
  }
}
```

### 5. Decorator Pattern (Decorador)

**Propósito:** Anexar responsabilidades adicionais a um objeto dinamicamente.

**Aplicação no Desafio:**

- Adicionar funcionalidades extras a pedidos (embalagem de presente, seguro, entrega expressa)
- Adicionar logging e métricas aos repositories

**Exemplo de Estrutura:**

```typescript
interface Order {
  calculate(): Money;
}

class BasicOrder implements Order {
  calculate(): Money {
    return this.subtotal;
  }
}

class GiftWrapDecorator implements Order {
  constructor(private order: Order) {}
  
  calculate(): Money {
    return this.order.calculate().add(new Money(5.00, 'BRL'));
  }
}

class InsuranceDecorator implements Order {
  constructor(private order: Order) {}
  
  calculate(): Money {
    const orderTotal = this.order.calculate();
    const insurance = orderTotal.multiply(0.02); // 2% de seguro
    return orderTotal.add(insurance);
  }
}
```

### 6. Chain of Responsibility (Cadeia de Responsabilidade)

**Propósito:** Evitar o acoplamento do remetente de uma solicitação ao seu receptor, dando a mais de um objeto a chance de tratar a solicitação.

**Aplicação no Desafio:**

- Pipeline de validações de pedido
- Cada validador pode aprovar, rejeitar ou passar para o próximo

**Exemplo de Estrutura:**

```typescript
interface OrderValidator {
  setNext(validator: OrderValidator): OrderValidator;
  validate(order: Order): ValidationResult;
}

class StockValidator implements OrderValidator {
  private next: OrderValidator | null = null;
  
  setNext(validator: OrderValidator): OrderValidator {
    this.next = validator;
    return validator;
  }
  
  validate(order: Order): ValidationResult {
    // Valida estoque
    if (!this.hasStock(order)) {
      return ValidationResult.fail("Produto sem estoque");
    }
    
    if (this.next) {
      return this.next.validate(order);
    }
    
    return ValidationResult.success();
  }
}

class PaymentValidator implements OrderValidator {
  // Similar ao StockValidator
}

class FraudValidator implements OrderValidator {
  // Similar ao StockValidator
}
```

### 7. Adapter Pattern (Adaptador)

**Propósito:** Converter a interface de uma classe em outra interface esperada pelos clientes.

**Aplicação no Desafio:**

- Adaptar diferentes APIs de pagamento para uma interface comum
- Adaptar diferentes formatos de entrada (REST, GraphQL, CLI)

**Exemplo de Estrutura:**

```typescript
// Interface unificada
interface PaymentGateway {
  charge(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult>;
}

// Adaptador para Stripe
class StripeAdapter implements PaymentGateway {
  constructor(private stripeClient: StripeSDK) {}
  
  async charge(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult> {
    // Adapta a chamada para o formato do Stripe
    const stripeResult = await this.stripeClient.charges.create({
      amount: amount.cents,
      currency: amount.currency.toLowerCase(),
      source: paymentMethod.token
    });
    
    // Converte resposta do Stripe para formato unificado
    return new PaymentResult(
      stripeResult.id,
      stripeResult.status === 'succeeded',
      stripeResult.failure_message
    );
  }
}
```

### 8. Command Pattern (Comando)

**Propósito:** Encapsular uma solicitação como um objeto, permitindo parametrizar clientes com diferentes solicitações e suportar operações reversíveis.

**Aplicação no Desafio:**

- Encapsular ações como objetos (CreateOrder, CancelOrder, RefundOrder)
- Facilitar auditoria e operações de undo/redo

**Exemplo de Estrutura:**

```typescript
interface Command {
  execute(): Promise<void>;
  undo(): Promise<void>;
}

class CreateOrderCommand implements Command {
  constructor(
    private orderRepository: OrderRepository,
    private orderData: CreateOrderDTO
  ) {}
  
  async execute(): Promise<void> {
    const order = Order.create(this.orderData);
    await this.orderRepository.save(order);
  }
  
  async undo(): Promise<void> {
    // Reverter a criação do pedido
  }
}

class CommandExecutor {
  private history: Command[] = [];
  
  async execute(command: Command): Promise<void> {
    await command.execute();
    this.history.push(command);
  }
  
  async undo(): Promise<void> {
    const command = this.history.pop();
    if (command) {
      await command.undo();
    }
  }
}
```

---

## 📁 Estrutura de Pastas Proposta

```
src/
├── domain/
│   ├── entities/
│   │   ├── Order.ts
│   │   ├── OrderItem.ts
│   │   ├── Customer.ts
│   │   ├── Payment.ts
│   │   └── Product.ts
│   │
│   ├── value-objects/
│   │   ├── Money.ts
│   │   ├── Email.ts
│   │   ├── OrderStatus.ts
│   │   ├── Address.ts
│   │   └── PaymentMethod.ts
│   │
│   ├── services/
│   │   ├── PricingService.ts
│   │   ├── OrderValidator.ts
│   │   └── DiscountCalculator.ts
│   │
│   └── events/
│       ├── OrderCreatedEvent.ts
│       ├── OrderPaidEvent.ts
│       └── OrderCancelledEvent.ts
│
├── application/
│   ├── ports/
│   │   ├── input/
│   │   │   ├── CreateOrderUseCase.ts
│   │   │   ├── CancelOrderUseCase.ts
│   │   │   ├── ProcessPaymentUseCase.ts
│   │   │   ├── GetOrderUseCase.ts
│   │   │   └── ListOrdersUseCase.ts
│   │   │
│   │   └── output/
│   │       ├── OrderRepository.ts
│   │       ├── ProductRepository.ts
│   │       ├── CustomerRepository.ts
│   │       ├── PaymentGateway.ts
│   │       ├── NotificationService.ts
│   │       └── EventPublisher.ts
│   │
│   └── use-cases/
│       ├── CreateOrderHandler.ts
│       ├── ProcessPaymentHandler.ts
│       ├── CancelOrderHandler.ts
│       └── GetOrderHandler.ts
│
└── infrastructure/
    ├── adapters/
    │   ├── input/
    │   │   ├── rest/
    │   │   │   ├── OrderController.ts
    │   │   │   ├── PaymentController.ts
    │   │   │   └── routes.ts
    │   │   │
    │   │   ├── graphql/
    │   │   │   ├── OrderResolver.ts
    │   │   │   └── schema.graphql
    │   │   │
    │   │   └── cli/
    │   │       └── OrderCommands.ts
    │   │
    │   └── output/
    │       ├── repositories/
    │       │   ├── PostgresOrderRepository.ts
    │       │   ├── MongoOrderRepository.ts
    │       │   ├── InMemoryOrderRepository.ts
    │       │   └── PostgresCustomerRepository.ts
    │       │
    │       ├── payment/
    │       │   ├── StripeAdapter.ts
    │       │   ├── PayPalAdapter.ts
    │       │   ├── PagSeguroAdapter.ts
    │       │   └── PaymentGatewayFactory.ts
    │       │
    │       ├── notification/
    │       │   ├── EmailNotificationService.ts
    │       │   ├── SmsNotificationService.ts
    │       │   ├── PushNotificationService.ts
    │       │   └── NotificationFactory.ts
    │       │
    │       └── events/
    │           ├── InMemoryEventPublisher.ts
    │           └── RabbitMQEventPublisher.ts
    │
    └── config/
        ├── DependencyInjection.ts
        ├── DatabaseConfig.ts
        └── ServerConfig.ts
```

---

## 📋 Requisitos Funcionais Detalhados

### 1. Criação de Pedidos

**RF-01:** O sistema deve permitir a criação de pedidos a partir de múltiplas fontes (API REST, GraphQL, CLI).

**RF-02:** Cada pedido deve conter: cliente, itens (produto, quantidade, preço unitário), endereço de entrega, método de pagamento.

**RF-03:** O sistema deve validar a disponibilidade de estoque antes de criar o pedido.

**RF-04:** O sistema deve calcular o valor total do pedido incluindo descontos, frete e impostos.

**RF-05:** O sistema deve aplicar regras de desconto baseadas em estratégias configuráveis.

### 2. Processamento de Pagamentos

**RF-06:** O sistema deve suportar múltiplos gateways de pagamento (Stripe, PayPal, PagSeguro).

**RF-07:** O sistema deve suportar diferentes métodos de pagamento (cartão de crédito, PIX, boleto).

**RF-08:** O sistema deve lidar com pagamentos assíncronos (callbacks de webhook).

**RF-09:** O sistema deve registrar todas as tentativas de pagamento para auditoria.

**RF-10:** O sistema deve permitir estornos e reembolsos parciais ou totais.

### 3. Gestão de Status

**RF-11:** O pedido deve transitar pelos seguintes status: Criado → Pagamento Pendente → Pago → Em Separação → Enviado → Entregue / Cancelado.

**RF-12:** Cada mudança de status deve gerar um evento que notifica observadores registrados.

**RF-13:** O sistema deve manter histórico de todas as mudanças de status com timestamp e usuário responsável.

### 4. Notificações

**RF-14:** O sistema deve enviar notificações por email quando o pedido for criado, pago, enviado e entregue.

**RF-15:** O sistema deve suportar múltiplos canais de notificação (email, SMS, push notification).

**RF-16:** As notificações devem ser enviadas de forma assíncrona para não bloquear o fluxo principal.

### 5. Consultas e Relatórios

**RF-17:** O sistema deve permitir buscar pedidos por ID, cliente, status e período.

**RF-18:** O sistema deve permitir listar todos os pedidos de um cliente com paginação.

**RF-19:** O sistema deve gerar relatórios de vendas por período, produto e categoria.

---

## 🔧 Requisitos Não-Funcionais

**RNF-01 - Testabilidade:** O domínio deve ser 100% testável sem dependências externas.

**RNF-02 - Desacoplamento:** A troca de qualquer adaptador não deve impactar o domínio ou outros adaptadores.

**RNF-03 - Extensibilidade:** Deve ser fácil adicionar novos gateways de pagamento, canais de notificação ou fontes de dados.

**RNF-04 - Performance:** O sistema deve processar pelo menos 100 pedidos por segundo.

**RNF-05 - Manutenibilidade:** O código deve seguir princípios SOLID e ter cobertura de testes acima de 80%.

**RNF-06 - Documentação:** Todas as interfaces públicas devem estar documentadas.

---

## 🎓 Níveis de Complexidade

### Nível 1: Básico (20-30 horas)

**Objetivos:**

1. Implementar entidade Order com validações básicas
2. Criar Repository Pattern com implementação in-memory
3. Implementar um adaptador REST simples para criar e buscar pedidos
4. Criar testes unitários para o domínio
5. Implementar Strategy Pattern para cálculo de descontos

**Entregáveis:**

- Entidades de domínio (Order, OrderItem, Customer)
- Value Objects (Money, Email, OrderStatus)
- Repository interface e implementação in-memory
- Controller REST básico
- Suite de testes unitários

### Nível 2: Intermediário (40-50 horas)

**Objetivos:**

Todos os objetivos do Nível 1, mais:

6. Adicionar persistência real (PostgreSQL ou MongoDB)
7. Implementar Factory Pattern para gateways de pagamento
8. Criar sistema de eventos com Observer Pattern
9. Implementar Chain of Responsibility para validações
10. Adicionar adaptador GraphQL

**Entregáveis:**

- Implementação de repository com banco real
- Factory para payment gateways
- Sistema de eventos e notificações
- Pipeline de validações
- API GraphQL funcional

### Nível 3: Avançado (60-80 horas)

**Objetivos:**

Todos os objetivos dos Níveis 1 e 2, mais:

11. Implementar Decorator Pattern para funcionalidades extras
12. Criar Command Pattern para operações auditáveis
13. Adicionar suporte a múltiplos adaptadores de entrada (REST + GraphQL + CLI)
14. Implementar sistema de cache
15. Adicionar métricas e observabilidade
16. Implementar testes de integração e E2E

**Entregáveis:**

- Sistema completo com todos os padrões
- CLI funcional
- Sistema de cache
- Dashboard de métricas
- Suite completa de testes (unitários, integração, E2E)
- Documentação completa da API

### Nível 4: Microserviços (100-120 horas)

**Objetivos:**

Todos os objetivos dos Níveis 1, 2 e 3, mais:

17. Decompor o monólito em microserviços independentes
18. Implementar comunicação assíncrona com message broker (RabbitMQ/Kafka)
19. Criar API Gateway para roteamento e autenticação
20. Implementar Service Discovery (Consul/Eureka) ou usar Kubernetes
21. Adicionar Circuit Breaker para resiliência
22. Implementar Saga Pattern para transações distribuídas
23. Configurar CQRS com Event Sourcing
24. Adicionar Distributed Tracing (Jaeger/Zipkin)
25. Implementar BFF (Backend for Frontend) para web e mobile
26. Configurar deploy com Docker Compose e Kubernetes

**Entregáveis:**

- 5-6 microserviços independentes (Order, Payment, Catalog, Customer, Notification, Shipping)
- Message broker configurado com filas e exchanges
- API Gateway com autenticação JWT
- Service mesh ou service discovery
- Saga implementation para fluxo de pedido
- Event store para event sourcing
- Distributed tracing configurado
- BFFs para diferentes clientes
- Manifests Kubernetes completos
- Documentação de arquitetura e ADRs (Architecture Decision Records)

---

## 🧪 Critérios de Avaliação

### Arquitetura (40 pontos)

- Separação clara entre camadas (Domain, Application, Infrastructure): 10 pontos
- Inversão de dependências corretamente aplicada: 10 pontos
- Domain independente de frameworks e bibliotecas externas: 10 pontos
- Uso apropriado de portas e adaptadores: 10 pontos

### Padrões de Projeto (30 pontos)

- Implementação correta de pelo menos 5 padrões: 15 pontos
- Padrões aplicados em contextos apropriados: 10 pontos
- Código limpo e seguindo princípios SOLID: 5 pontos

### Testes (20 pontos)

- Cobertura de testes acima de 80%: 10 pontos
- Testes unitários isolados e rápidos: 5 pontos
- Testes de integração cobrindo fluxos principais: 5 pontos

### Código e Documentação (10 pontos)

- Código legível e bem organizado: 5 pontos
- Documentação clara de interfaces e use cases: 3 pontos
- README com instruções de setup e execução: 2 pontos

---

## 🚀 Desafios Bônus (Além dos Microserviços)

### Service Mesh Avançado

Implementar service mesh completo com Istio ou Linkerd:

- Mutual TLS automático entre serviços
- Traffic splitting para canary deployments
- Fault injection para chaos engineering
- Observabilidade avançada com métricas detalhadas

### Multi-tenancy

Suportar múltiplos clientes (tenants) na mesma infraestrutura:

- Isolamento de dados por tenant
- Configurações personalizadas por tenant
- Rate limiting por tenant
- Billing e métricas por tenant

### GraphQL Federation

Unificar APIs de múltiplos serviços em um único schema GraphQL:

- Schema stitching entre microserviços
- Resolvers distribuídos
- Subscriptions em tempo real
- DataLoader para otimização de queries

### Serverless Functions

Migrar alguns componentes para funções serverless:

- AWS Lambda/Azure Functions para processamento de eventos
- Step Functions para orquestração de workflows
- API Gateway gerenciado
- DynamoDB Streams para event sourcing

### Machine Learning Integration

Adicionar capacidades de ML ao sistema:

- Recomendação de produtos baseada em histórico
- Detecção de fraude em tempo real
- Previsão de demanda para gestão de estoque
- Chatbot para atendimento ao cliente

---

## 📚 Referências e Recursos

### Livros Recomendados

**"Clean Architecture" - Robert C. Martin:** Fundamentos de arquitetura limpa e princípios SOLID.

**"Domain-Driven Design" - Eric Evans:** Conceitos de DDD que complementam arquitetura hexagonal.

**"Design Patterns" - Gang of Four:** Catálogo clássico de padrões de projeto.

**"Patterns of Enterprise Application Architecture" - Martin Fowler:** Padrões para aplicações corporativas, incluindo Repository.

### Artigos e Tutoriais

**"Hexagonal Architecture" - Alistair Cockburn:** Artigo original sobre o padrão.

**"The Clean Architecture" - Robert C. Martin:** Visão sobre arquitetura limpa e camadas.

**"SOLID Principles" - Uncle Bob:** Explicação detalhada dos princípios SOLID.

### Ferramentas Úteis

**TypeScript:** Para type safety e melhor experiência de desenvolvimento.

**Jest:** Framework de testes para JavaScript/TypeScript.

**Docker:** Para containerização e facilitar setup do ambiente.

**PostgreSQL/MongoDB:** Bancos de dados para persistência.

**Stripe/PayPal SDKs:** Para integração real com gateways de pagamento.

---

## 📝 Checklist de Implementação

### Fase 1: Fundação

- [ ] Criar estrutura de pastas seguindo arquitetura hexagonal
- [ ] Implementar entidades de domínio (Order, OrderItem, Customer)
- [ ] Criar value objects (Money, Email, OrderStatus)
- [ ] Definir interfaces das portas (use cases e repositories)
- [ ] Escrever testes unitários para o domínio

### Fase 2: Casos de Uso

- [ ] Implementar CreateOrderUseCase
- [ ] Implementar ProcessPaymentUseCase
- [ ] Implementar CancelOrderUseCase
- [ ] Implementar GetOrderUseCase
- [ ] Adicionar validações com Chain of Responsibility

### Fase 3: Adaptadores de Saída

- [ ] Implementar OrderRepository (in-memory)
- [ ] Implementar OrderRepository (PostgreSQL/MongoDB)
- [ ] Criar PaymentGatewayFactory
- [ ] Implementar adaptadores para gateways de pagamento
- [ ] Implementar serviços de notificação

### Fase 4: Adaptadores de Entrada

- [ ] Criar REST API com controllers
- [ ] Adicionar validação de entrada
- [ ] Implementar tratamento de erros
- [ ] Adicionar documentação da API (Swagger/OpenAPI)

### Fase 5: Padrões Avançados

- [ ] Implementar Strategy para descontos
- [ ] Adicionar Observer para eventos
- [ ] Criar Decorators para funcionalidades extras
- [ ] Implementar Command Pattern para auditoria

### Fase 6: Qualidade e Testes

- [ ] Adicionar testes de integração
- [ ] Configurar CI/CD
- [ ] Adicionar métricas e logging
- [ ] Realizar code review e refatoração

### Fase 7: Microserviços (Opcional - Nível 4)

- [ ] Identificar bounded contexts e definir fronteiras dos serviços
- [ ] Extrair Order Service como primeiro microserviço
- [ ] Extrair Payment Service
- [ ] Extrair Catalog/Inventory Service
- [ ] Extrair Customer Service
- [ ] Extrair Notification Service
- [ ] Configurar message broker (RabbitMQ/Kafka)
- [ ] Implementar comunicação assíncrona entre serviços
- [ ] Criar API Gateway (Kong/AWS API Gateway)
- [ ] Implementar Service Discovery ou configurar Kubernetes
- [ ] Adicionar Circuit Breaker em todas as integrações
- [ ] Implementar Saga Pattern para fluxo de pedido
- [ ] Configurar CQRS com Event Sourcing
- [ ] Adicionar Distributed Tracing (Jaeger/Zipkin)
- [ ] Criar BFFs para web e mobile
- [ ] Configurar Docker Compose para desenvolvimento
- [ ] Criar manifests Kubernetes para produção
- [ ] Configurar auto-scaling e health checks
- [ ] Implementar zero-downtime deployments
- [ ] Documentar arquitetura e criar ADRs

---

## 💡 Dicas de Implementação

### Comece Simples

Não tente implementar todos os padrões de uma vez. Comece com o fluxo básico de criação de pedido e vá adicionando complexidade gradualmente.

### Teste Primeiro

Escreva testes antes de implementar. Isso garante que seu código seja testável e ajuda a pensar no design.

### Evite Over-engineering

Nem todo problema precisa de todos os padrões. Use o bom senso e aplique padrões onde eles realmente agregam valor.

### Mantenha o Domínio Puro

O código do domínio não deve ter dependências de frameworks, bibliotecas de I/O ou infraestrutura. Use apenas linguagem pura.

### Use Injeção de Dependências

Configure um container de DI para facilitar a montagem dos objetos e inversão de dependências.

### Documente Decisões

Mantenha um arquivo ADR (Architecture Decision Records) documentando decisões importantes de arquitetura.

### Para Microserviços: Comece com Monólito Modular

Não comece direto com microserviços! Construa primeiro um monólito bem estruturado com arquitetura hexagonal. Migre para microserviços apenas quando houver uma necessidade clara de negócio.

### Para Microserviços: Defina Bounded Contexts

Use conceitos de Domain-Driven Design para identificar fronteiras naturais entre serviços. Cada microserviço deve ter responsabilidade única e coesa.

### Para Microserviços: Priorize Comunicação Assíncrona

Sempre que possível, use mensageria ao invés de chamadas síncronas. Isso reduz acoplamento e aumenta resiliência.

### Para Microserviços: Pense em Observabilidade desde o Início

Adicione logging estruturado, métricas e tracing desde o primeiro serviço. É muito mais difícil adicionar depois.

---

## 🎯 Métricas de Sucesso

### Cobertura de Testes

- Domínio: 100%
- Application: 90%+
- Infrastructure: 70%+

### Acoplamento

- Domínio não deve importar nada de Application ou Infrastructure
- Application pode importar apenas de Domain
- Infrastructure pode importar de Domain e Application

### Performance

**Monólito:**
- Criação de pedido: < 200ms
- Consulta de pedido: < 100ms
- Processamento de pagamento: < 2s

**Microserviços (Nível 4):**
- End-to-end latency: < 500ms (P95)
- Inter-service latency: < 50ms (P95)
- Message processing: < 100ms (P95)
- Service availability: > 99.9%

### Manutenibilidade

- Complexidade ciclomática: < 10 por método
- Tamanho de classes: < 300 linhas
- Tamanho de métodos: < 50 linhas

### Microserviços - Métricas Adicionais (Nível 4)

**Independência de Deploy:**
- Cada serviço pode ser deployado independentemente
- Zero downtime durante deploys
- Rollback em < 5 minutos

**Resiliência:**
- Circuit breaker funcionando corretamente
- Degradação graciosa quando serviços dependentes estão offline
- Retry com backoff exponencial implementado

**Observabilidade:**
- Traces completos de ponta a ponta com correlation IDs
- Métricas RED (Rate, Errors, Duration) para todos os serviços
- Logs estruturados agregados centralmente
- Dashboards com KPIs de negócio e técnicos

---

## 🤝 Próximos Passos

Após completar o desafio (incluindo o Nível 4 de Microserviços, se escolhido), você pode evoluir o sistema com:

1. **Service Mesh Avançado:** Implementar Istio ou Linkerd para traffic management, security e observability avançada

2. **Multi-Cloud:** Deploy em múltiplos provedores cloud (AWS, Azure, GCP) com failover automático

3. **Edge Computing:** Adicionar CDN e edge functions para reduzir latência global

4. **Machine Learning Ops:** Integrar modelos de ML para recomendações, detecção de fraude e forecasting

5. **Blockchain Integration:** Para casos de uso que necessitem imutabilidade e auditoria distribuída

6. **Real-time Analytics:** Stream processing com Apache Flink ou Kafka Streams para análises em tempo real

---

## 📞 Conclusão

Este desafio foi projetado para consolidar conhecimentos em arquitetura de software, padrões de projeto e microserviços através de um caso prático e realista. 

**A jornada proposta:**

1. **Fundação Sólida (Níveis 1-3):** Construir um monólito modular bem estruturado com arquitetura hexagonal e padrões clássicos
2. **Evolução Natural (Nível 4):** Quando justificável, decompor em microserviços mantendo os mesmos princípios de design
3. **Aprendizados Práticos:** Entender que arquitetura não é sobre escolher monólito OU microserviços, mas saber quando e como fazer a transição

A arquitetura hexagonal é a base perfeita para ambos os mundos: permite criar um monólito testável e evolutivo, e facilita enormemente a migração para microserviços quando necessário, pois os serviços já nascem com portas e adaptadores bem definidos.

**Lembre-se:** 
- O objetivo não é apenas fazer funcionar, mas criar um código que seja fácil de entender, modificar e testar
- Microserviços não são uma solução mágica - eles trazem complexidade operacional significativa
- Comece simples, evolua com propósito, sempre guiado por necessidades reais de negócio

Boa sorte com o desafio!

---

**Documento criado em:** Janeiro de 2026  
**Versão:** 2.0 (Atualizado com Microserviços)  
**Autor:** Claude (Anthropic)