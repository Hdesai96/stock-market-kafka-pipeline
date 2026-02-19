# Real-Time Stock Market Data Pipeline

A real-time data engineering pipeline that streams stock market data using **Apache Kafka** on **AWS EC2**, stores it in **AWS S3**, and enables SQL analytics via **AWS Glue** and **Amazon Athena**.

---

## Architecture

```
CSV Data → Kafka Producer → EC2 Kafka Broker → Kafka Consumer → AWS S3 → Glue Crawler → Athena
```

![Pipeline Architecture](https://img.shields.io/badge/Apache%20Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=white)
![AWS](https://img.shields.io/badge/Amazon_AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Apache Kafka 3.3.1 | Real-time message streaming |
| AWS EC2 | Kafka broker hosting |
| AWS S3 | Data lake storage |
| AWS Glue | Schema detection & cataloging |
| Amazon Athena | SQL querying on S3 |
| Python (kafka-python, s3fs) | Producer & Consumer scripts |
| Jupyter Notebook | Development environment |

---

## Getting Started

### Prerequisites
- AWS Account with EC2, S3, Glue, Athena access
- Python 3.x
- Jupyter Notebook

### Installation

```bash
pip install kafka-python s3fs pandas
```

### 1. Launch EC2 & Start Kafka

```bash
# SSH into EC2
ssh -i "your-key.pem" ec2-user@your-ec2-public-ip

# Start ZooKeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka (new terminal)
bin/kafka-server-start.sh config/server.properties

# Create Topic
bin/kafka-topics.sh --create --topic stock-market --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1
```

### 2. Configure server.properties

```properties
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://YOUR_EC2_PUBLIC_IP:9092
```

### 3. Run the Producer

```python
from kafka import KafkaProducer
from json import dumps
import pandas as pd
from time import sleep

producer = KafkaProducer(
    bootstrap_servers=['YOUR_EC2_IP:9092'],
    value_serializer=lambda x: dumps(x).encode('utf-8')
)

df = pd.read_csv('indexProcessed.csv')

while True:
    dict_stock = df.sample(1).to_dict(orient="records")[0]
    producer.send('project1', value=dict_stock)
    sleep(1)
```

### 4. Run the Consumer (saves to S3)

```python
from kafka import KafkaConsumer
from json import loads
from s3fs import S3FileSystem
import json

consumer = KafkaConsumer(
    'project1',
    bootstrap_servers=['YOUR_EC2_IP:9092'],
    auto_offset_reset='earliest',
    value_deserializer=lambda x: loads(x.decode('utf-8'))
)

s3 = S3FileSystem()

for count, i in enumerate(consumer):
    with s3.open("s3://your-bucket/stock_market_{}.json".format(count), 'w') as file:
        json.dump(i.value, file)
```

### 5. Set Up Glue Crawler

1. Go to **AWS Glue → Crawlers → Create Crawler**
2. Set S3 path: `s3://your-bucket/`
3. Create/select an IAM role
4. Set output database: `stock_market_db`
5. Run the crawler

### 6. Query with Athena

```sql
SELECT * FROM stock_market_db.your_table LIMIT 10;
```

---

## Project Structure

```
├── producer.ipynb        # Kafka Producer notebook
├── consumer.ipynb        # Kafka Consumer notebook
├── indexProcessed.csv    # Stock market dataset
└── README.md
```

---

## AWS Security Group Rules

| Type | Port | Source |
|------|------|--------|
| SSH | 22 | 0.0.0.0/0 |
| Custom TCP (Kafka) | 9092 | 0.0.0.0/0 |
| Custom TCP (ZooKeeper) | 2181 | 0.0.0.0/0 |

---

## Dataset

The project uses `indexProcessed.csv` — a stock market index dataset streamed row-by-row to simulate real-time data ingestion.

---

## Key Learnings

- Setting up Apache Kafka on cloud infrastructure (EC2)
- Building real-time data pipelines with Python
- Configuring Kafka broker networking (`advertised.listeners`)
- Integrating Kafka with AWS services (S3, Glue, Athena)
- Querying data lake files using serverless SQL

---

## Author

**Desai**  
Student | Aspiring Data Engineer  

---

## Note

Remember to **stop your EC2 instance** when not in use to avoid AWS charges.
