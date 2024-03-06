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

Now let's create a JsonConsumer in our example folder starting with this outline borrowed from the JsonProducer file swapping out ProducerConfig with ConsumerConfig.

```java
package org.example;
  
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.example.data.Ride;

import java.time.temporal.ChronoUnit;
import java.time.*;
import java.util.List;
import java.util.Properties;

import io.confluent.kafka.serializers.KafkaJsonDeserializerConfig;

public class JsonConsumer {
	// make private
	private Properties props = new Properties();
	private KafkaConsumer<String, Ride> consumer;

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

		props.put(ConsumerConfig.GROUP_ID_CONFIG, "java-group-1");
		props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
		// new consumer
		consumer = new KafkaConsumer<String, Ride>(props);
		
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
	System.out.println("Consuming form kafka started");
	while(true){
		var results = consumer.poll(Duration.of(1, ChronoUnit.SECONDS));
		for(ConsumerRecord<String, Ride> result: results) {
			System.out.println(result.value().DOLocationID);
		}
	}
}
```

And then we call this method in the main method:

```java
JsonConsumer jsonConsumer = new JsonConsumer();
jsonConsumer.consumeFromKafka();
```

# Kafka Configuration
We see in the previous section that there is a lot of terminology that is involved in configuring Kafka. Here we will dive into the theory behind all of this.

**What is a Kafka cluster?** Just nodes or machines running Kafka that are talking to each other in a network using some communication protocol. Earlier on, **zookeeper** was used for communication, zookeeper was used for topics (what are the topics exist/ have been created, the partitions or retention for the topic, etc.). Now Kafka uses Kafka's internals and so the topic itself stores all of that information and that is what is used for the nodes to communication with each other.

**What is a topic?** A topic is just a sequence of events coming in. An event / message has a key, value, and timestamp (type long) and these are released by our Kafka Producer.

**How does a cluster provide reliability in Kafka?** A topic exists on one of the clusters (let's say Node 1 of nodes 0, 1, 2). If Node 1 goes down, the cluster rebalances and there is only Node 0 and Node 2 but the topic itself is gone. Then all the producers that produce to that topic will stop and consumers will also stop consuming from that topic. This is obviously bad, so this is where the concept of **replication** comes in. The topic is replicated in some number of nodes, for example let's say there are two nodes that contain a copy of the topic. In this case, one of the nodes is designated a leader and the others are followers. The log for the message will be saved in the leader node and duplicated in the follower nodes. The producers and consumers will talk to the leader. Now if the leader dies, the producers and consumers won't notice a difference (maybe a delay) as they will instead then directly connect to a follower. Then that follower node becomes a leader and since the cluster knew it wanted a replication of 2 for that topic then it will select one of the other nodes to become a follower so that replication still exists.

**What is retention?** How long data will be retained by Kafka. Since nodes are limited in memory, we don't want to keep the messages forever. The retention deletes anything that is older than whatever you have set for the retention. This is easy with logs because we are always appending so we find the first message that is past the retention period and get rid of everything after that.

**What is a partition?** This is what allows Kafka to scale. First let's talk about how data is stored: if we are partitioning our topic, we are taking our topic and dividing it into different parts (number of parts indicated by partition number) which are stored on different nodes (or possibly the same due to duplication). The reason why we use partitions is because let's say 
a consumer group was consuming from a topic that had a partition size of 1. Now if the messages start to get really large, the consumer within the consumer group may need more time to read the message and during that time new messages had already been generated. So it would make sense to have a duplicate consumer within that group who is still reading from the topic while the other is trying to process the message (so essentially half the messages go to consumer 1 and the other half to consumer 2). But for a topic with partition of size 1, only one consumer within that consumer group can read from it. So if instead we have a topic with partition size 2 then there are two nodes that each have half the messages. Then we can have duplicate consumers with still a one-to-one connection between topics and consumers in a particular consumer group. Now how does Kafka know these consumers belong to the same group? That is the **consumer group ID**. If consumers have the same consumer group ID, then Kafka will understand they belong to the same entity. You may have more consumers than partitions and that would allow Kafka to redirect messages from a partition to a new consumer when one goes down.

**What are offsets?** Offsets allow Kafka to know which messages it needs to send to consumers. There is an offset attached to each topic, you can think of it as starting at 0 with the first message and incrementing by 1. When your consumer consumes from a particular topic, it also tells Kafka that it has consumed the message. It will go to the Kafka broker that it has committed offset 10 and it will store this information inside an internal Kafka topic \_\_consumer\_offset. So if a consumer dies and reconnects to Kafka using the consumer group ID and then Kafka will know the number of messages the consumers in that group ID will have already committed. The key for that \_\_consumer\_offset topic is <consumer.group.id, topic, partition, offset> and it uses that information to tell the consumer how to proceed. Within offsets there is an option AUTO.OFFSET.RESET which can take on two values: latest (default) or earliest. This is how Kafka should react when a new group ID is attached to it. When a new group ID is attached and the "latest" option is set, it will only send new messages to the new group ID. If "earliest" is set, then all the messages starting with the earliest message stored will be sent to the new group ID.

**What was ACK.ALL?** This is from the producer side, and it stands for **acknowledgement all**. There are different options for ACK.ALL: 0 - (fire and forget) means it sends a message to the leader node and it doesn't check that the message was delivered to the reader, 1 (leader successful) so the message has to be converted into the log of the leader before it returns success to the producer, all (leader + follower successful) the producer waits for the leader and all followers have written the message to their respective logs before sending a success to the producer. If you are using all, then at least one delivery will be guaranteed since even if a leader goes down the follower definitely has the message. But this is slow. Which one you want to use depends on how sure you want to be that your messages are delivered.

But this is not it! There are more configurations that Kafka provides. Feel free to go through the documentation on the kafka website.

# Kafka Streams Basics

In this example, we will see how keys play an important. role when messages are outputted through Kafka.

Let's start by making a new class called JsonKStream which will be a Kafka Stream application. We have rides coming in from our topic and we are going to group these by key, which is the PULocationID. Then we count it and sent the count over to a new topic called rides-pulocation-count.

```java
// imports
package org.example;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.Topology;
import org.apache.kafka.streams.kstream.Consumed;
import org.apache.kafka.streams.kstream.Produced;
import org.example.customserdes.CustomSerdes;
import org.apache.kafka.common.serialization.Deserializer;
import org.apache.kafka.common.serialization.Serde;
import org.apache.kafka.common.serialization.Serializer;
import io.confluent.kafka.serializers.KafkaJsonDeserializer;
import io.confluent.kafka.serializers.KafkaJsonSerializer;
import org.example.data.Ride;

import java.util.Map;
import java.util.HashMap;
import java.util.Properties;

public class JsonKStream {

	private Properties props = new Properties();
	
	public JsonKStream() {
		String userName = System.getenv("CLUSTER_API_KEY");
		String passWord = System.getenv("CLUSTER_API_SECRET");
		String bootstrapServer = System.getenv("BOOTSTRAP_SERVER");
		// Streams Config here
		props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServer);
		props.put("security.protocol", "SASL_SSL");
		props.put("sasl.jaas.config", String.format("org.apache.kafka.common.security.plain.PlainLoginModule required username='%s' password='%s';", userName, passWord));
		props.put("sasl.mechanism", "PLAIN");
		props.put("client.dns.lookup", "use_all_dns_ips");
		props.put("session.timeout.ms", "45000");
	}

  

	public void countPLocation() {
	// blank for now
	}

  

	private Serde<Ride> getJsonSerde() {

		Map<String, Object> serdeProps = new HashMap<>();
		serdeProps.put("json.value.type", Ride.class);
		final Serializer<Ride> mySerializer = new KafkaJsonSerializer();
		mySerializer.configure(serdeProps, false);
		
		final Deserializer<Ride> myDeserializer = new KafkaJsonDeserializer<>();
		myDeserializer.configure(serdeProps, false);
		
		return Serdes.serdeFrom(mySerializer, myDeserializer);
	}

public static void main(String[] args) {

	// blank for now

	}

}
```

What we are seeing in the getJsonSerde method is the creation of a SerDe which stands for serializer/deserializer. A **serializer** refers to a component responsible for converting data from its native format (such as Java objects) into a format that can be transmitted and stored in Kafka topics. Similarly, a **deserializer** converts Kafka messages back into their native formats.

When we create the serde we create a hash map from the Ride class. Then we tell it to use a json serializer/deserializer.

We need to add a few more properties:

```java
// like a group ID
props.put(StreamsConfig.APPLICATION_ID_CONFIG, "kafka_tutorial.kstream.count.plocation.v1");
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest");
props.put(StreamsConfig.CACHE_MAX_BYTES_BUFFERING_CONFIG, 0);
```

And let's fill in our countPLocation method:

```java
public void countPLocation() {

	StreamsBuilder streamsBuilder = new StreamsBuilder();
	// returns a kafka stream
	var ridesStream = streamsBuilder.stream("rides", Consumed.with(Serdes.String(), getJsonSerde()));
	var puLocationCount = ridesStream.groupByKey().count().toStream();
	puLocationCount.to("rides-pulocation-count", Produced.with(Serdes.String(), Serdes.Long()));

}
```

In Confluent Cloud, go ahead and create the rides-pulocation-count topic.

We need to add a few more things to our countPLocation method:
```java
var kStreams = new KafkaStreams(streamsBuilder.build(), props);
kStreams.start();

Runtime.getRuntime().addShutdownHook(new Thread(kStreams::close));
```

And let's update our main function to include the following:
```java
var object = new JsonKStream();
object.countPLocation();
```

We updated our publishRides method in the JsonProducer class to the following:

```java
public void publishRides(List<Ride> rides) throws ExecutionException, InterruptedException {
	KafkaProducer<String, Ride> kafkaProducer = new KafkaProducer<String, Ride>(props);
	for(Ride ride: rides) {
		ride.tpep_pickup_datetime = LocalDateTime.now().minusMinutes(20);
		ride.tpep_dropoff_datetime = LocalDateTime.now();
		var record = kafkaProducer.send(new ProducerRecord<>("rides", String.valueOf(ride.PULocationID), ride), (metadata, exception) -> {
			if(exception != null) {
				System.out.println(exception.getMessage());
			}
		});
		Thread.sleep(500);
	}
}
```

Now when we have both the JsonProducer and the JsonKStream going we can see that the rides-pulocation-count topic is being sent messages.

In this example, we have 1 app that is receiving messages from both partitions. In the case where we have two apps, each partition will send messages to a unique app. So then the counts for the individual apps might be wrong if the partitions don't have all the messages for a given key. So how does Kafka solve this problem? When the producer is writing to Kafka (e.g., our rides topic) it is writing to different partitions and it will **hash** the key and **modulo** it by the partition count to determine which partition it should be sent to. That means the producer makes sure that a partition receives all messages for a given key. When the key is null then it will just round-robin the message throughout the different partitions. This way the data sizes are always equal for each partition (assuming there is an equal number of events for each key).

# Kafka Stream Joins

We will set up two topics where the data can be joined and we will build a topology (Kafka stream application) to do the join.

Recall that the rides topic key was the Drop-off location ID. In Kafka, you can only do joins on the key of a message. So we will create another topic, called the pickup-location, with the same key but different message (this one will send the pickup location). Then we will join these on their keys (PU location ID) with a Kafka stream application.

Create a new class called JsonKStreamJoins with the same Streams configuration as our JsonKStreams class.

