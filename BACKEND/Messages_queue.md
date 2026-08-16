# 05 — Message Queues & Event-Driven Architecture

## Kafka, RabbitMQ, Pub/Sub — 25+ Interview Questions

---

### RabbitMQ

```javascript
const amqplib = require('amqplib');

// Producer
async function publishMessage(exchange, routingKey, message) {
    const connection = await amqplib.connect('amqp://localhost');
    const channel = await connection.createChannel();
    await channel.assertExchange(exchange, 'topic', { durable: true });
    channel.publish(exchange, routingKey, Buffer.from(JSON.stringify(message)), { persistent: true });
    await channel.close();
    await connection.close();
}

// Consumer
async function consumeMessages(exchange, pattern, handler) {
    const connection = await amqplib.connect('amqp://localhost');
    const channel = await connection.createChannel();
    await channel.assertExchange(exchange, 'topic', { durable: true });
    const { queue } = await channel.assertQueue('', { exclusive: true });
    await channel.bindQueue(queue, exchange, pattern);
    channel.prefetch(1);
    channel.consume(queue, async (msg) => {
        try {
            await handler(JSON.parse(msg.content.toString()));
            channel.ack(msg);
        } catch (err) {
            channel.nack(msg, false, true);
        }
    });
}
```

---

### Kafka

```javascript
const { Kafka } = require('kafkajs');

const kafka = new Kafka({ clientId: 'my-app', brokers: ['localhost:9092'] });

// Producer
async function produce(topic, messages) {
    const producer = kafka.producer();
    await producer.connect();
    await producer.send({ topic, messages: messages.map(m => ({ key: m.key, value: JSON.stringify(m.value) })) });
    await producer.disconnect();
}

// Consumer
async function consume(topic, groupId, handler) {
    const consumer = kafka.consumer({ groupId });
    await consumer.connect();
    await consumer.subscribe({ topic, fromBeginning: false });
    await consumer.run({ eachMessage: async ({ message }) => await handler(JSON.parse(message.value.toString())) });
}
```

---

### Dead Letter Queue

```javascript
await channel.assertQueue('orders', {
    durable: true,
    arguments: {
        'x-dead-letter-exchange': 'dlx',
        'x-dead-letter-routing-key': 'orders.failed',
        'x-message-ttl': 60000
    }
});
```

---

### Interview Questions (25+)

**Q1: When would you use a message queue?**
A: "Async processing, service decoupling, load leveling. Examples: sending emails, processing uploads, distributing work. Don't use when you need immediate response."

**Q2: Kafka vs RabbitMQ?**
A: "Kafka: high-throughput event streaming, replay capability, log storage. RabbitMQ: task queues, lower latency, traditional messaging. Kafka processes millions/sec; RabbitMQ is simpler."

**Q3: What is a Dead Letter Queue?**
A: "Queue for messages that fail processing. After max retries, message moves to DLQ. Monitor DLQ for patterns. Alert on growth. Manually process or requeue."

**Q4: What is idempotency in message processing?**
A: "Same message processed multiple times = same result. Important for at-least-once delivery. Implement with unique message IDs, database deduplication."

**Q5: What's the difference between pub/sub and point-to-point?**
A: "Pub/sub: one message, multiple subscribers (event notification). Point-to-point: one message, one consumer (task distribution). Use pub/sub for broadcasting; point-to-point for work distribution."

**Q6: What is message ordering?**
A: "Kafka: ordered within partition. RabbitMQ: ordered within queue. Use partition key for ordering in Kafka. Handle out-of-order messages in consumer."

**Q7: How do you handle message failures?**
A: "Retry with exponential backoff. Dead letter queue after max retries. Circuit breaker for downstream services. Idempotent processing for safe retries."

**Q8: What is at-least-once vs exactly-once delivery?**
A: "At-least-once: message delivered, maybe duplicates. Exactly-once: message delivered once (complex). Most systems use at-least-once + idempotency."

**Q9: What is message serialization?**
A: "Convert message to bytes for transmission. JSON (human-readable), Avro (schema evolution), Protocol Buffers (efficient). Choose based on compatibility and performance needs."

**Q10: How do you monitor message queues?**
A: "Track queue depth, consumer lag, processing time. Monitor dead letter queue growth. Tools: RabbitMQ Management, Kafka Manager, Prometheus."

**Q11: What is consumer group in Kafka?**
A: "Group of consumers that share the work. Each partition assigned to one consumer in group. Adding consumers beyond partitions is wasteful. Enables parallel processing."

**Q12: What is message backpressure?**
A: "When consumers can't keep up with producers. Solutions: scale consumers, increase partition count, implement rate limiting, buffer messages."

**Q13: What is saga pattern?**
A: "Distributed transaction across services using compensating actions. Orchestration: central coordinator. Choreography: events between services. Handle failures with rollback."

**Q14: How do you test message queue consumers?**
A: "Unit test message handler logic. Integration test with real queue. Contract test message format. Load test consumer throughput."

**Q15: What is event sourcing?**
A: "Store events instead of current state. Replay events to rebuild state. Audit trail built-in. Complex queries need CQRS. Used with message queues."

**Q16: What is CQRS?**
A: "Command Query Responsibility Segregation. Separate write model (commands) from read model (queries). Optimize each independently. Often combined with event sourcing."

**Q17: How do you handle message schema evolution?**
A: "Use schema registry (Confluent). Add fields, don't remove. Use default values. Version schemas. Test backward compatibility."

**Q18: What is message priority?**
A: "High priority messages processed first. RabbitMQ supports priority queues. Kafka doesn't natively (use separate topics). Implement with multiple queues."

**Q19: How do you implement request-reply pattern?**
A: "Correlation ID to match request and reply. Reply queue per request or shared with correlation. Set timeout for replies. Handle missing replies gracefully."

**Q20: What is message deduplication?**
A: "Prevent processing same message twice. Use unique message ID. Store processed IDs in database or Redis with TTL. Implement in consumer."

**Q21: How do you handle large messages?**
A: "Store payload in object storage (S3). Send reference in message. Compress messages. Split into chunks. Use appropriate message size limits."

**Q22: What is transactional outbox pattern?**
A: "Write to database and outbox table in same transaction. Poll outbox and publish to queue. Ensures consistency between database and message queue."

**Q23: How do you scale message consumers?**
A: "Horizontal scaling with consumer groups. Increase partition count (Kafka). Add more queues (RabbitMQ). Ensure idempotent processing."

**Q24: What is message routing?**
A: "Direct messages to specific queues based on rules. RabbitMQ: exchange types (direct, topic, fanout). Kafka: partition key. Implement with routing keys."

**Q25: How do you handle message ordering guarantees?**
A: "Kafka: use partition key for ordering. RabbitMQ: single consumer per queue. Handle out-of-order in consumer with sequence numbers or buffering."

---

### Complete Kafka Implementation

```javascript
const { Kafka, logLevel } = require('kafkajs');

const kafka = new Kafka({
    clientId: 'my-app',
    brokers: ['localhost:9092'],
    logLevel: logLevel.WARN,
    retry: { initialRetryTime: 100, retries: 8 }
});

// Producer with batching
class KafkaProducer {
    constructor() {
        this.producer = kafka.producer({
            maxInFlightRequests: 1,
            idempotent: true,
            transactionalId: 'my-app-producer'
        });
    }
    
    async connect() {
        await this.producer.connect();
    }
    
    async send(topic, messages) {
        return this.producer.send({
            topic,
            messages: messages.map(m => ({
                key: m.key,
                value: JSON.stringify(m.value),
                headers: m.headers || {}
            }))
        });
    }
    
    async sendBatch(topic, messages) {
        return this.producer.sendBatch({
            topicMessages: [{
                topic,
                messages: messages.map(m => ({
                    key: m.key,
                    value: JSON.stringify(m.value)
                }))
            }]
        });
    }
}

// Consumer with consumer groups
class KafkaConsumer {
    constructor(groupId) {
        this.consumer = kafka.consumer({ groupId });
    }
    
    async subscribe(topic, handler) {
        await this.consumer.connect();
        await this.consumer.subscribe({ topic, fromBeginning: false });
        
        await this.consumer.run({
            eachBatchAutoResolve: true,
            eachBatch: async ({ batch, resolveOffset, heartbeat }) => {
                for (const message of batch.messages) {
                    try {
                        const value = JSON.parse(message.value.toString());
                        await handler(value, {
                            topic: batch.topic,
                            partition: batch.partition,
                            offset: message.offset,
                            timestamp: message.timestamp
                        });
                        resolveOffset(message.offset);
                    } catch (error) {
                        console.error('Message processing failed:', error);
                        // Implement dead letter queue
                    }
                }
                await heartbeat();
            }
        });
    }
}
```

---

### Complete RabbitMQ Implementation

```javascript
const amqplib = require('amqplib');

class RabbitMQClient {
    constructor(url) {
        this.url = url;
        this.connection = null;
        this.channel = null;
    }
    
    async connect() {
        this.connection = await amqplib.connect(this.url);
        this.channel = await this.connection.createChannel();
        
        this.connection.on('error', (err) => {
            console.error('RabbitMQ connection error:', err);
            setTimeout(() => this.connect(), 5000);
        });
    }
    
    async publish(exchange, routingKey, message, options = {}) {
        await this.channel.assertExchange(exchange, 'topic', { durable: true });
        this.channel.publish(exchange, routingKey, Buffer.from(JSON.stringify(message)), {
            persistent: true,
            contentType: 'application/json',
            ...options
        });
    }
    
    async consume(exchange, pattern, handler) {
        await this.channel.assertExchange(exchange, 'topic', { durable: true });
        const { queue } = await this.channel.assertQueue('', { exclusive: true });
        await this.channel.bindQueue(queue, exchange, pattern);
        
        this.channel.prefetch(1);
        
        await this.channel.consume(queue, async (msg) => {
            try {
                const content = JSON.parse(msg.content.toString());
                await handler(content, msg.properties);
                this.channel.ack(msg);
            } catch (error) {
                console.error('Message processing failed:', error);
                this.channel.nack(msg, false, true);
            }
        });
    }
    
    async close() {
        if (this.channel) await this.channel.close();
        if (this.connection) await this.connection.close();
    }
}
```

---

### Additional Interview Questions (15+)

**Q26: How do you implement exactly-once delivery?**
A: "Idempotent consumers + transactional outbox. Kafka: idempotent producer + consumer offset management. RabbitMQ: publisher confirms + manual ack. Most systems use at-least-once + idempotency."

**Q27: What is Kafka consumer lag?**
A: "Difference between latest message produced and last message consumed. Monitor with Kafka tools. High lag indicates consumer can't keep up. Solutions: scale consumers, optimize processing."

**Q28: How do you handle message ordering in distributed systems?**
A: "Kafka: partition key for ordering within partition. RabbitMQ: single consumer per queue. For global ordering: use single partition (limits throughput). Handle out-of-order with sequence numbers."

**Q29: What is transactional outbox pattern?**
A: "Write to database and outbox table in same transaction. Poll outbox and publish to queue. Ensures consistency. Use CDC (Change Data Capture) for efficiency."

**Q30: How do you implement message retry with backoff?**
A: "Exponential backoff: 1s, 2s, 4s, 8s. Add jitter to prevent thundering herd. Use delayed message exchange (RabbitMQ) or retry topic (Kafka). Max retry count before DLQ."

**Q31: What is Kafka Streams?**
A: "Client library for stream processing. Process data in real-time. Stateful operations (windowing, joins). Exactly-once semantics. Use for real-time analytics."

**Q32: How do you handle message schema evolution?**
A: "Schema registry (Confluent). Backward/forward compatibility. Add fields with defaults. Remove fields carefully. Version schemas."

**Q33: What is fan-out pattern?**
A: "One message to multiple queues/topics. Each consumer processes independently. Use for notifications, event broadcasting. RabbitMQ: fanout exchange. Kafka: multiple consumer groups."

**Q34: How do you implement request-reply pattern?**
A: "Correlation ID in message. Reply queue (per request or shared). Set timeout. Handle missing replies. Use temporary queues for replies."

**Q35: What is priority queue in RabbitMQ?**
A: "Messages with higher priority processed first. Set x-max-priority in queue declaration. Priority 0-255. Use for urgent messages."

**Q36: How do you handle message compression?**
A: "Kafka: built-in compression (gzip, snappy, lz4, zstd). RabbitMQ: application-level compression. Choose based on CPU vs bandwidth trade-off."

**Q37: What is Kafka exactly-once semantics?**
A: "Idempotent producer + transactional API. Consumer read_committed isolation. End-to-end exactly-once. Complex but possible."

**Q38: How do you monitor message queues?**
A: "Queue depth, consumer lag, processing time, error rate. Tools: Kafka Manager, RabbitMQ Management, Prometheus, Grafana. Alert on anomalies."

**Q39: What is message partitioning?**
A: "Split messages across partitions for parallel processing. Kafka: partition key determines partition. Enable horizontal scaling. Handle ordering within partition."

**Q40: How do you handle poison messages?**
A: "Messages that always fail processing. After max retries, move to DLQ. Alert on DLQ growth. Manual inspection and requeue. Don't retry indefinitely."

---

*Next: [06 — Database Mastery](06-Databases.md)*
