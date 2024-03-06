In this module we are going to answer the following questions:
* What is stream processing?
* What is Kafka and how does it play a role in stream processing?
We will discuss some of the message properties of stream processing, configurations specific to Kafka (partition, replication, retention), time surrounding in stream processing, Kafka consumers and producers, how data is partitioned inside stream processing, how to work with Kafka streams (both in Java and spark streaming Python examples). We will also talk about schema and how it plays a role in stream processing, Kafka connect, and the kasql database.

# What is stream processing?
Let's first talk about data exchange. Data exchange can happen over multiple sources, generally in the form of APIs. One computer is streaming some sort of information/data and that data is exchanged.

Let's start with a basic example of data exchange in the real-world. Think of a bulletin board of flyers. A **producer** is someone who goes to the bulletin board and attaches a flyer that they want the world to see (data exchange). On the other side, **data consumers** or users pass by the bulletin board and take whatever action they see necessary after seeing what is posted. We can complicate this data exchange a bit more. Let's say as a data consumer I am interested in certain topics, for example mathematics. Then the producer will attach their flyer to a certain topic, say real analysis, and then only the people who are subscribed to that topic will receive the flyer. This is more similar to how data is exchanged in stream processing. In stream processing, the data is exchanged in real-time. Let's say the producer produces some data to the real analysis topic. Then the real analysis topic immediately receives that topic and sends it to its subscribers. This exchange happens with maybe a few seconds of delay, but generally very low latency as compared to batch processing.

# What is Kafka?
Here we will talk about how we can we take our bulletin board example and make Kafka our central streaming architecture. Our bulletin board that is backed by Kafka will have several topics. A **topic** is just a continuous stream of **events** which can be thought of as a single data point at a certain timestamp. The collection of these events go into the topic which are then read by the consumer. In Kafka, when you store data you store them as **logs**, that is logs are how events are stored in a topic. Each event contains a **message**, in Kafka a message has three structures: (1) key, (2) value, (3) timestamp. The key is used for determining what the key is for the message (partitioning) the value is the data exchange you actually want to do, and the timestamp is when the message was created. Kafka provides robustness/reliability to the topic, meaning that even if the servers/nodes are down then you will still receive your data (replication). Adds flexibility so that topics can be small or big, many or few consumers. Kafka also provides scalability. There is a retention / expiration on top of topics which allows them to sustain messages for a longer period of time. Once a consumer reads the message, it doesn't mean that the other consumers can't retrieve it anymore.

## What is the need for stream processing in the real world?
We used to have an architecture of monoliths which talk to a central database. They used to be generally really big codes and this caused some issues. The trend these days is to work towards microservices (think of now as many data sources). In a microservice architecture, we have many small applications. They do need to talk to each other (maybe through APIs, message pass, central database). This works as long as your data size isn't that large, but with more microservices and increasing data you need a streaming service which allows microservices to communicate to each other. So a microservice writes to a Kafka topic (which are in terms of events). Then another microservice can read the messages from the topics. Kafka also allows microservices to work closely with a database (CDC, change data capture part of Kafka Connect). In CDC the database writes to a Kafka topic and then the microservices read from the topics.

# Confluent Cloud
Confluent Cloud will allow us to have a Kafka Cluster that is connected to Confluent Cloud. It is free for 30-days and we can connect it to our Google Cloud account. Create a cluster in Confluent Cloud that will be connected to GCP. Now we will create an API key.
![[Screenshot 2024-03-04 at 5.38.44 PM.png]]

Now let's create a topic called tutorial_topic with 2 partitions:
![[Screenshot 2024-03-04 at 5.40.15 PM.png]]

In the advanced settings, keep retention to 1 day then Save & Create.

Now toggle to Messages at the top and let's produce a new message (go ahead and produce the default message).

![[Screenshot 2024-03-04 at 5.42.19 PM.png]]

Now if we click on the message we can see the key, value, and timestamp.
![[Screenshot 2024-03-04 at 5.43.37 PM.png]]

Now let's create a dummy connector (Datagen Source).
![[Screenshot 2024-03-04 at 5.45.24 PM.png]]

Set the topic to tutorial_topic and output record format to JSON and the ORDERS template. We see that it is working:
![[Screenshot 2024-03-04 at 5.50.12 PM.png]]
Now pause this connector so that it doesn't use up all your credits.

# Kafka producer consumer
In this video, we are going to produce messages programmatically to Kafka using the taxi rides data. We are going to use Java as a programming language since Kafka libraries are well-maintained for Java and Python isn't as well-maintained.

Let's create a rides topic in Confluent Cloud with 2 partitions with a retention time of 1 day.

Navigate to the Clients tab and create a new client that will be written in Java.

![[Screenshot 2024-03-04 at 6.31.51 PM.png]]

In our repo, we will create a folder for week_6_stream_processing and one called kafka_examples. We see that we need this build.gradle file from Confluent Cloud:
![[Screenshot 2024-03-04 at 6.50.07 PM.png]]
We will add these two to the dependences:
```gradle
implementation 'com.opencsv:opencsv:5.7.1'
implementation 'io.confluent:kafka-json-serializer:7.3.1'
```
Now we will create a new directory:
![[Screenshot 2024-03-04 at 6.52.04 PM.png]]

Now in this folder let's create a subfolder called data. In it, we will include this Java file, called a constructor, which just takes in each data point as a string and parses all of the fields using the respective data types:

```java
package org.example.data;

import java.nio.DoubleBuffer;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Ride {
    public Ride(String[] arr) {
        VendorID = arr[0];
        tpep_pickup_datetime = LocalDateTime.parse(arr[1], DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        tpep_dropoff_datetime = LocalDateTime.parse(arr[2], DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        passenger_count = Integer.parseInt(arr[3]);
        trip_distance = Double.parseDouble(arr[4]);
        RatecodeID = Long.parseLong(arr[5]);
        store_and_fwd_flag = arr[6];
        PULocationID = Long.parseLong(arr[7]);
        DOLocationID = Long.parseLong(arr[8]);
        payment_type = arr[9];
        fare_amount = Double.parseDouble(arr[10]);
        extra = Double.parseDouble(arr[11]);
        mta_tax = Double.parseDouble(arr[12]);
        tip_amount = Double.parseDouble(arr[13]);
        tolls_amount = Double.parseDouble(arr[14]);
        improvement_surcharge = Double.parseDouble(arr[15]);
        total_amount = Double.parseDouble(arr[16]);
        congestion_surcharge = Double.parseDouble(arr[17]);
    }
    public Ride(){}
    public String VendorID;
    public LocalDateTime tpep_pickup_datetime;
    public LocalDateTime tpep_dropoff_datetime;
    public int passenger_count;
    public double trip_distance;
    public long RatecodeID;
    public String store_and_fwd_flag;
    public long PULocationID;
    public long DOLocationID;
    public String payment_type;
    public double fare_amount;
    public double extra;
    public double mta_tax;
    public double tip_amount;
    public double tolls_amount;
    public double improvement_surcharge;
    public double total_amount;
    public double congestion_surcharge;

}
```


Now in the example folder we will create a new file called JsonProducer.Java. Now he seemed to be working off a template, not sure where that came from:

```java
package org.example;

import com.opencsv.CSVReader;
import com.opencsv.exceptions.CsvException;
import org.example.data.Ride;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.util.List;
import java.util.Properties;
import java.util.concurrent.ExecutionException;
import java.util.stream.Collectors;

import org.apache.kafka.clients.producer.*;
import org.apache.kafka.streams.StreamsConfig;

public class JsonProducer {

	public List<Ride> getRides() throws IOException, CsvException {
		// pulls from resources folder in main
		var ridesStream = this.getClass().getResource("/rides.csv");
		var reader = new CSVReader(new FileReader(ridesStream.getFile()));
		reader.skip(1);
		return reader.readAll().stream().map(arr -> new Ride(arr))
			.collect(Collectors.toList());
	}
	
	public static void main(String[] args) throws IOException, CsvException, ExecutionException, InterruptedException {
		var producer = new JsonProducer();
		var rides = producer.getRides();
	}
	
}
```

The code above seems to just be reading off of a CSV file. Now we will create a method:

```java
Properties props = new Properties();

public JsonProducer(){
	// read in environment variables
	String userName = System.getenv("CLUSTER_API_KEY");
	String passWord = System.getenv("CLUSTER_API_SECRET");
	
	// coming froming Confluent Cloud Configuration snippet
	props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "pkc-12576z.us-west2.gcp.confluent.cloud:9092");
	props.put("security.protocol", "SASL_SSL");
	props.put("sasl.jaas.config", String.format("org.apache.kafka.common.security.plain.PlainLoginModule required username='%s' password='%s';", userName, passWord));
	props.put("sasl.mechanism", "PLAIN");
	props.put("session.timeout.ms", "45000");
	props.put(ProducerConfig.ACKS_CONFIG, "all");
	props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
	props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "io.confluent.kafka.serializers.KafkaJsonSerializer");
}

public void publishRides(List<Ride> rides) throws ExecutionException, InterruptedException {
	var kafkaProducer = new KafkaProducer<String, Ride>(props);
	for(Ride ride: rides) {
		kafkaProducer.send(new ProducerRecord<>("rides", String.valueOf(ride.PULocationID), ride);
		Thread.sleep(500);
	}
}
```

Since we don't want to expose our API credentials, you can export CLUSTER_API_KEY to be the username from that key .txt file we downloaded and the CLUSTER_API_SECRET similarly.

Then we can add the following to our main function:

```java
producer.publishRides(rides);
```

In order to run this in VSCode, you need to tell VSCode that this kafka_examples directory is a Java Project.

Now let's create a JsonConsumer in our data folder starting with this outline borrowed from the JsonProducer file swapping out ProducerConfig with ConsumerConfig.

```java
package org.example;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.producer.ProducerConfig;

import java.util.Properties;

public class JsonConsumer {
	// make private
	private Properties props = new Properties();

	public JsonConsumer(){
		// read in environment variables
		String userName = System.getenv("CLUSTER_API_KEY");
		String passWord = System.getenv("CLUSTER_API_SECRET");
		
		// coming froming Confluent Cloud Configuration snippet
		props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "pkc-12576z.us-west2.gcp.confluent.cloud:9092");
		props.put("security.protocol", "SASL_SSL");
		props.put("sasl.jaas.config", String.format("org.apache.kafka.common.security.plain.PlainLoginModule required username='%s' password='%s';", userName, passWord));
		props.put("sasl.mechanism", "PLAIN");
		props.put("session.timeout.ms", "45000");
		// ACKS removed for Consumer
		
		// de-serialization for Consumer
		props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
		props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "io.confluent.kafka.serializers.KafkaJsonDeserializer");

		props.put(ConsumerConfig.GROUP_ID_CONFIG, "kafka_tutorial_example.jsonconsumer");
		// subscribe to the topic
		consumer.subscribe(List.of("rides"));
	}
	public static void main(String[] args) {

	}

}
```

Now that we have our properties set up, let's set up our consumer method in the JsonConsumer class:

```java
private KafkaConsumer<String, Ride> consumer;

public void consumeFromKafka() {
	var results = consumer.poll(Duration.of(1, ChronoUnit.SECONDS));
	do {
		for(ConsumerRecord<string, Ride> result: results) {
			System.out.println(result.value().DOLocationID);
		}
		results = consumer.poll(Duration.of(1, ChronoUnit.SECONDS));
	}
	while(!results.isEmpty());
}
```

And then we call this method in the main method:

```java
JsonConsumer jsonConsumer = new JsonConsumer();
jsonConsumer.consumeFromKafka();
```