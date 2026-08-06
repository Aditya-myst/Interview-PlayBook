# 05 — Message Queues

## Kafka, RabbitMQ — Async Processing

---

### Why Message Queues?

Without queues: Services call each other directly (synchronous, tightly coupled).
With queues: Services communicate through messages (asynchronous, decoupled).

```
Without Queue (Synchronous):
Order Service → Payment Service → Inventory Service → Email Service
[User waits for ALL operations to complete]

With Queue (Asynchronous):
Order Service → Queue → Payment Service
                     → Inventory Service
                     → Email Service
[User gets immediate response, processing happens in background]
```

---

### Benefits of Message Queues

| Benefit | Description |
|---------|-------------|
| **Decoupling** | Services don't need to know about each other |
| **Scalability** | Add more consumers to handle load |
| **Reliability** | Messages persist until processed |
| **Async Processing** | User gets immediate response |
| **Load Leveling** | Buffer spikes in traffic |
| **Retry** | Failed messages can be retried |

---

### RabbitMQ

A traditional message broker implementing AMQP protocol.

```
Producer → Exchange → Queue → Consumer

Exchange types:
├── Direct: Route by exact routing key
├── Fanout: Broadcast to all queues
├── Topic: Route by pattern (user.*)
└── Headers: Route by message headers
```

#### RabbitMQ Example (Node.js)

```javascript
const amqplib = require('amqplib');

// Producer
async function publishMessage(queue, message) {
    const connection = await amqplib.connect('amqp://localhost');
    const channel = await connection.createChannel();
    
    await channel.assertQueue(queue, { durable: true });
    channel.sendToQueue(queue, Buffer.from(JSON.stringify(message)), {
        persistent: true
    });
    
    console.log('Message sent:', message);
}

// Consumer
async function consumeMessages(queue, handler) {
    const connection = await amqplib.connect('amqp://localhost');
    const channel = await connection.createChannel();
    
    await channel.assertQueue(queue, { durable: true });
    channel.prefetch(1);  // Process one message at a time
    
    channel.consume(queue, async (msg) => {
        const content = JSON.parse(msg.content.toString());
        try {
            await handler(content);
            channel.ack(msg);  // Acknowledge success
        } catch (err) {
            channel.nack(msg);  // Negative acknowledge - retry
        }
    });
}

// Usage
// Order Service
await publishMessage('order.created', { orderId: 123, userId: 456 });

// Payment Service
consumeMessages('order.created', async (order) => {
    await processPayment(order);
    await publishMessage('payment.completed', { orderId: order.orderId });
});

// Email Service
consumeMessages('payment.completed', async (data) => {
    await sendConfirmationEmail(data.orderId);
});
```

---

### Apache Kafka

A distributed event streaming platform. Different from traditional message queues.

```
Producer → Topic (Partitioned) → Consumer Group

Topic: user-events
├── Partition 0: [msg1, msg4, msg7]
├── Partition 1: [msg2, msg5, msg8]
└── Partition 2: [msg3, msg6, msg9]

Consumer Group A:
├── Consumer 1 → Partition 0
├── Consumer 2 → Partition 1
└── Consumer 3 → Partition 2
```

#### Kafka vs RabbitMQ

| Aspect | Kafka | RabbitMQ |
|--------|-------|----------|
| **Model** | Log (append-only) | Queue (delete after consume) |
| **Replay** | ✓ (read old messages) | ✗ |
| **Ordering** | Per partition | Per queue |
| **Throughput** | Very high (millions/sec) | Moderate |
| **Latency** | Higher | Lower |
| **Use case** | Event streaming, logs | Task queues, RPC |

#### Kafka Example (Node.js)

```javascript
const { Kafka } = require('kafkajs');

const kafka = new Kafka({
    clientId: 'my-app',
    brokers: ['localhost:9092']
});

// Producer
async function produce(topic, messages) {
    const producer = kafka.producer();
    await producer.connect();
    
    await producer.send({
        topic,
        messages: messages.map(msg => ({
            key: msg.key,
            value: JSON.stringify(msg.value)
        }))
    });
    
    await producer.disconnect();
}

// Consumer
async function consume(topic, groupId, handler) {
    const consumer = kafka.consumer({ groupId });
    await consumer.connect();
    await consumer.subscribe({ topic, fromBeginning: false });
    
    await consumer.run({
        eachMessage: async ({ topic, partition, message }) => {
            const value = JSON.parse(message.value.toString());
            await handler(value, { topic, partition, offset: message.offset });
        }
    });
}

// Usage
// Producer
await produce('user-events', [
    { key: 'user-123', value: { type: 'signup', userId: 123 } },
    { key: 'user-456', value: { type: 'signup', userId: 456 } }
]);

// Consumer
consume('user-events', 'email-service', async (event) => {
    if (event.type === 'signup') {
        await sendWelcomeEmail(event.userId);
    }
});
```

---

### Pub/Sub Pattern

Publisher sends messages to a topic; all subscribers receive them.

```
Publisher → Topic → Subscriber 1
                 → Subscriber 2
                 → Subscriber 3
```

#### Redis Pub/Sub

```javascript
const redis = require('redis');

// Publisher
const publisher = redis.createClient();
await publisher.publish('notifications', JSON.stringify({
    type: 'new_order',
    orderId: 123
}));

// Subscriber
const subscriber = redis.createClient();
await subscriber.subscribe('notifications', (message) => {
    const data = JSON.parse(message);
    console.log('Received:', data);
});
```

---

### Message Patterns

#### 1. Point-to-Point (Queue)
One message processed by one consumer.

```
Producer → Queue → Consumer (only one gets it)
```

#### 2. Pub/Sub (Topic)
One message received by all subscribers.

```
Publisher → Topic → Subscriber 1
                 → Subscriber 2
                 → Subscriber 3
```

#### 3. Request/Reply
Producer expects a response.

```
Service A → Queue → Service B
Service A ← Reply Queue ← Service B
```

---

### Dead Letter Queue (DLQ)

Messages that fail processing go to DLQ for inspection.

```javascript
// RabbitMQ DLQ setup
await channel.assertQueue('orders', {
    durable: true,
    arguments: {
        'x-dead-letter-exchange': 'dlx',
        'x-dead-letter-routing-key': 'orders.failed',
        'x-message-ttl': 60000  // Max time in queue
    }
});

// Process DLQ messages
consumeMessages('orders.failed', async (failedMessage) => {
    console.log('Failed message:', failedMessage);
    // Alert, log, or manually retry
});
```

---

### At-Least-Once vs Exactly-Once Delivery

| Guarantee | Description | Trade-off |
|-----------|-------------|-----------|
| **At-most-once** | Message may be lost | Fast, no retries |
| **At-least-once** | Message delivered, maybe duplicates | Safe, need idempotency |
| **Exactly-once** | Message delivered exactly once | Complex, expensive |

**Most systems use at-least-once + idempotency.**

```javascript
// Idempotent message handler
const processedMessages = new Set();

async function handleMessage(message) {
    if (processedMessages.has(message.id)) {
        console.log('Already processed, skipping');
        return;
    }
    
    await processOrder(message);
    processedMessages.add(message.id);
}
```

---

### Interview Questions

**Q: What's the difference between a message queue and a pub/sub system?**

A: "Message queue: point-to-point, one message processed by one consumer (task distribution). Pub/sub: one message delivered to all subscribers (event notification). Use queues for work distribution; pub/sub for broadcasting events."

**Q: When would you use Kafka vs RabbitMQ?**

A: "Kafka: high throughput event streaming, log aggregation, replay capability, stream processing. RabbitMQ: task queues, request/reply, lower latency, simpler setup. Kafka for data pipelines; RabbitMQ for traditional message brokering."

**Q: How do you handle failed messages?**

A: "Retry with exponential backoff. After max retries, send to Dead Letter Queue (DLQ). Monitor DLQ for patterns. Implement idempotent handlers to safely retry. Alert on DLQ growth."

**Q: What's idempotency and why is it important?**

A: "An operation is idempotent if performing it multiple times has the same effect as once. Important because message queues may deliver duplicates (at-least-once delivery). Implement idempotency using unique message IDs, database constraints, or deduplication tables."

---

*Next: [06 — Databases Deep Dive](06-Databases.md)*
