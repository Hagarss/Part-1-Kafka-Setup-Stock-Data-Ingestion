
# Project Report – Week 1: Kafka Setup & Stock Data Ingestion

**Real-Time Stock Volatility Insight Pipeline with Contextual Enrichment**

## Week 1 Focus:
Set up Kafka and simulate streaming stock trade data into a Kafka topic.

---

## Objective

The goal of this phase is to establish the foundation of the real-time data pipeline by:

- Installing and configuring Apache Kafka using Docker
- Creating a Kafka topic to stream stock trade data
- Developing a Python producer to simulate stock trades
- Streaming messages in real time into Kafka

---

## Technologies Used

| Component              | Tool / Version                        |
|------------------------|----------------------------------------|
| Kafka                  | Apache Kafka 7.4.0 (via Docker)        |
| Zookeeper              | Confluent Zookeeper 7.4.0 (via Docker) |
| Messaging              | `kafka-python` Python library          |
| Development Language   | Python 3.10+                           |
| Deployment Environment | Docker Desktop on Windows             |

---

## Kafka Setup Details

### Docker Compose Configuration

Kafka and Zookeeper were deployed locally using Docker Compose to simplify setup and isolate dependencies.

**docker-compose.yml** configuration:
```yaml
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  kafka:
    image: confluentinc/cp-kafka:7.4.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
````

Kafka was started using:

```bash
docker-compose up -d
```

---

### Kafka Topic Creation

A topic called `stock_prices` was created for streaming stock trade messages:

```bash
docker exec -it stock-volatility-project-kafka-1 kafka-topics \
--create --topic stock_prices --bootstrap-server localhost:9092 \
--partitions 1 --replication-factor 1
```

**Result:**

```
Created topic stock_prices.
```

---

## Python Producer Script

A Python script (`producer.py`) was created to send simulated stock trade messages into the `stock_prices` topic on Kafka.

**Features:**

* Sends one message per second
* Randomly generates stock symbols, prices, and volumes
* Sends JSON-encoded messages using `kafka-python`

**Code:**

```python
from kafka import KafkaProducer
import json
import time
import random
from datetime import datetime

# Set up Kafka producer
producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

# Simulate some stock symbols
symbols = ['AAPL', 'TSLA', 'GOOGL', 'AMZN']

while True:
    # Create a fake stock message
    message = {
        'timestamp': datetime.utcnow().isoformat(),
        'symbol': random.choice(symbols),
        'price': round(random.uniform(100, 500), 2),
        'volume': random.randint(10, 500)
    }

    print(f"Sending: {message}")
    # Send message to Kafka topic
    producer.send('stock_prices', value=message)
    time.sleep(1)  # Wait 1 second before sending the next message

```

**Output:**


![image](https://github.com/user-attachments/assets/9a5f7e8b-42b1-499e-a8f0-5cf5c0b8effb)


---

## Example Kafka Message

```json
{
  "timestamp": "2025-07-07T01:23:45Z",
  "symbol": "GOOGL",
  "price": 399.75,
  "volume": 420
}
```
![image](https://github.com/user-attachments/assets/563131c5-0614-4358-bc11-67334225f2eb)

---

## Week 1 Achievements

* [x] Installed Docker and Docker Compose
* [x] Successfully ran Kafka and Zookeeper containers
* [x] Created Kafka topic `stock_prices`
* [x] Developed and tested a real-time stock producer in Python
* [x] Validated message delivery to Kafka

---

## Next Steps (Week 2 Preview)

* Consume messages from Kafka using PySpark
* Enrich messages with weather data from OpenWeatherMap API
* Store enriched data in Delta Lake
* Begin integration of pipeline orchestration tools

---

