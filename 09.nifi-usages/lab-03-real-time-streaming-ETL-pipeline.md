# Lab: Data Streaming ETL pipeline with Kafka and NiFi

- Build a real-time data streaming ETL pipeline with NiFi and Kafka.

## Tasks

1. Configuration of Kafka producer and consume messages with NiFi. (Hard)
2. Build the dataflow. (Hard)

## Prerequisites

- Completion of the [Introduction to Kafka lab](../07.kafka-stream-processing/lab-introduction-to-kafka.md)

- Completion of the [Introduction to NiFi lab](../08.nifi-dataflow/lab-introduction-to-nifi.md) and [NiFi ETL pipeline](./lab-02-build-ETL-pipeline.md).

## Build consumer in NiFi

1. Use the "ConsumeKafkaRecord_2_6" processor to consume Kafka messages. Configure the processor with the following values:

- Kafka Brokers: `kafka01:9092,kafka02:9092`
- Topic Name(s): `demo-nyc-taxi-fare`
- Value Record Reader: `JsonTreeReader`
- Record Value Writer: `JsonRecordSetWriter`
- Group ID: `nifi-group`

2. Configure 2 Json controllers. Note that "Schema Access Strategy" should be "Use 'Schema Text' Property" and provide an avro schema for the controller.

## Build transformation process

1. Transform the data as needed, using at least 3 processors to complete this step.

## Insert into Postgres database

1. Run the following command to launch a Postgres shell:

   ```bash
   psql -U <postgres username> -d dsti_db
   ```

2. Create the target table for data insertion. Please make sure the table schema matches your data.

## Start real-time streaming from the kafka-data-producer container

1. Run the `kafka_producer.py` Python script inside the container to simulate real-time data, and continusly produce messages to the Kafka topic.

    ```bash
    docker exec -it kafka-data-producer /bin/sh
    # Inside the container:
    python producer.py \
        --brokers kafka01:9092,kafka02:9092 \
        --topic demo-nyc-taxi-fare \
        --csv data.csv \
        --delay 1
    ```

2. Check for any errors in the NiFi UI.

3. Verify that the data appears correctly in the Postgres table.