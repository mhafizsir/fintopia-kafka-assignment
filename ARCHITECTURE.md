# Kafka-Based Microservices Architecture

## Data Flow Architecture

```mermaid
graph TB
    %% External Layer
    Client[Client/User] -->|POST /api/orders| OrderProducer[Order Producer Service<br/>Port 8081<br/>Idempotent Producer]
    
    %% Kafka Layer
    OrderProducer -->|Kafka Producer| OrdersTopic[orders-topic<br/>3 partitions<br/>replication: 2]
    OrdersTopic --> OrderConsumer[Order Consumer Service<br/>Port 8082]
    OrderConsumer -->|Logs operations| LogsTopic[logs-topic<br/>Log compaction<br/>7-day TTL]
    
    %% Stream Processing
    OrdersTopic --> KafkaStreams[Kafka Streams App<br/>Port 8084<br/>Hourly Aggregation]
    KafkaStreams --> HourlyTopic[hourly-transaction-topic]
    
    %% Data Storage
    OrderConsumer --> MySQL[(MySQL Database<br/>transactions table)]
    
    %% Kafka Connect Layer
    HourlyTopic --> JDBCConnector[JDBC Sink Connector]
    JDBCConnector --> HourlySummary[(MySQL<br/>hourly_summary table)]
    
    LogsTopic --> ElasticConnector[Elasticsearch Sink Connector]
    ElasticConnector --> ElasticIndex[(Elasticsearch<br/>shopstream-logs index)]
    
    %% Monitoring Layer
    HourlySummary --> Kibana[Kibana Dashboard<br/>Port 5601]
    ElasticIndex --> Kibana
    
    %% Styling
    classDef service fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef kafka fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef database fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef connector fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef monitoring fill:#fff8e1,stroke:#f57f17,stroke-width:2px
    
    class OrderProducer,OrderConsumer,KafkaStreams service
    class OrdersTopic,LogsTopic,HourlyTopic kafka
    class MySQL,HourlySummary,ElasticIndex database
    class JDBCConnector,ElasticConnector connector
    class Kibana monitoring
```

## Component Architecture

```mermaid
graph LR
    %% Input Layer
    API[REST API<br/>/api/orders] --> Producer[Order Producer<br/>Spring Boot]
    
    %% Message Layer
    Producer --> Topic1[orders-topic]
    Producer --> Topic2[logs-topic]
    
    %% Processing Layer
    Topic1 --> Consumer[Order Consumer<br/>Spring Boot]
    Topic1 --> Streams[Kafka Streams<br/>Aggregation]
    
    %% Output Layer
    Consumer --> DB1[(MySQL<br/>transactions)]
    Streams --> Topic3[hourly-transaction-topic]
    
    %% Integration Layer
    Topic3 --> Connect1[Kafka Connect<br/>JDBC Sink]
    Topic2 --> Connect2[Kafka Connect<br/>Elasticsearch Sink]
    
    %% Storage Layer
    Connect1 --> DB2[(MySQL<br/>hourly_summary)]
    Connect2 --> ES[(Elasticsearch<br/>shopstream-logs)]
    
    %% Monitoring Layer
    DB2 --> Dashboard[Kibana Dashboard]
    ES --> Dashboard
    
    classDef api fill:#ffeb3b,stroke:#f57f17,stroke-width:2px
    classDef service fill:#2196f3,stroke:#0d47a1,stroke-width:2px
    classDef topic fill:#ff9800,stroke:#e65100,stroke-width:2px
    classDef storage fill:#9c27b0,stroke:#4a148c,stroke-width:2px
    classDef connect fill:#4caf50,stroke:#1b5e20,stroke-width:2px
    classDef monitor fill:#f44336,stroke:#b71c1c,stroke-width:2px
    
    class API api
    class Producer,Consumer,Streams service
    class Topic1,Topic2,Topic3 topic
    class DB1,DB2,ES storage
    class Connect1,Connect2 connect
    class Dashboard monitor
```

## Technology Stack

```mermaid
graph TB
    subgraph "Application Layer"
        SpringBoot[Spring Boot 3.4.5<br/>Java 21]
        REST[REST API<br/>JSON Processing]
    end
    
    subgraph "Message Streaming"
        Kafka[Apache Kafka 7.4.0<br/>Multi-broker Cluster]
        KStreams[Kafka Streams<br/>Real-time Processing]
        KConnect[Kafka Connect<br/>Data Integration]
    end
    
    subgraph "Data Storage"
        MySQL[MySQL 8.0<br/>Transactional Data]
        Elasticsearch[Elasticsearch 8.8.0<br/>Log Storage & Search]
    end
    
    subgraph "Monitoring & Ops"
        Kibana[Kibana 8.8.0<br/>Visualization]
        Docker[Docker Compose<br/>Container Orchestration]
    end
    
    SpringBoot --> Kafka
    Kafka --> KStreams
    Kafka --> KConnect
    KConnect --> MySQL
    KConnect --> Elasticsearch
    MySQL --> Kibana
    Elasticsearch --> Kibana
    
    classDef app fill:#e3f2fd,stroke:#1976d2
    classDef stream fill:#fff3e0,stroke:#f57c00
    classDef data fill:#f3e5f5,stroke:#7b1fa2
    classDef ops fill:#e8f5e8,stroke:#388e3c
    
    class SpringBoot,REST app
    class Kafka,KStreams,KConnect stream
    class MySQL,Elasticsearch data
    class Kibana,Docker ops
```

## Data Schema

```mermaid
erDiagram
    ORDER {
        string orderId PK
        string customerId
        string productId
        int quantity
        decimal price
        timestamp orderTime
        string status
    }
    
    TRANSACTION {
        bigint id PK
        string orderId UK
        string customerId
        string productId
        int quantity
        decimal price
        timestamp orderTime
        string status
        timestamp createdAt
    }
    
    HOURLY_SUMMARY {
        bigint id PK
        string hourWindow UK
        bigint transactionCount
        timestamp createdAt
        timestamp updatedAt
    }
    
    LOG_MESSAGE {
        timestamp timestamp
        string serviceName
        string status
        string errorMessage
    }
    
    ORDER ||--|| TRANSACTION : "processed_to"
    TRANSACTION ||--o{ HOURLY_SUMMARY : "aggregated_to"
    ORDER ||--o{ LOG_MESSAGE : "logs_operation"
    TRANSACTION ||--o{ LOG_MESSAGE : "logs_operation"
```

## Key Features Implemented

- ✅ **Idempotent Producer**: Ensures exactly-once semantics
- ✅ **Topic Configuration**: Proper partitioning and replication
- ✅ **Stream Processing**: Real-time hourly aggregation
- ✅ **Data Integration**: Kafka Connect with JDBC and Elasticsearch sinks
- ✅ **Error Handling**: Comprehensive logging and monitoring
- ✅ **Scalability**: Multi-broker Kafka cluster setup
- ✅ **Containerization**: Docker Compose for easy deployment