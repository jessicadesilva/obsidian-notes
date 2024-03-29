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

Recall that the rides topic key was the Drop-off location ID. In Kafka, you can only do joins on the key of a message. So we will create another topic, called the pickup-location, with the same key but different message (this one will send the pickup location). Then we will join these on their keys (PU location ID) with a Kafka stream application. Note that when you are going to join two topics, they need to have the same partition count in order for the keys to match.

First, let's create the second producer which will send out the messages for the rides_location topic (both using the class below and also setting it up in Confluent Cloud).

```java
package org.example;

import com.opencsv.exceptions.CsvException;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.example.data.PickupLocation;

import java.io.IOException;
import java.time.LocalDateTime;
import java.util.Properties;
import java.util.concurrent.ExecutionException;

public class JsonProducerPickupLocation {
    private Properties props = new Properties();

    public JsonProducerPickupLocation() {
        String userName = System.getenv("CLUSTER_API_KEY");
	String passWord = System.getenv("CLUSTER_API_SECRET");
	String bootstrapServer = System.getenv("BOOTSTRAP_SERVER");
	props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServer);
	props.put("security.protocol", "SASL_SSL");
	props.put("sasl.jaas.config", String.format("org.apache.kafka.common.security.plain.PlainLoginModule required username='%s' password='%s';", userName, passWord));
        props.put("sasl.mechanism", "PLAIN");
        props.put("client.dns.lookup", "use_all_dns_ips");
        props.put("session.timeout.ms", "45000");
        props.put(ProducerConfig.ACKS_CONFIG, "all");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "io.confluent.kafka.serializers.KafkaJsonSerializer");
    }

    public void publish(PickupLocation pickupLocation) throws ExecutionException, InterruptedException {
        KafkaProducer<String, PickupLocation> kafkaProducer = new KafkaProducer<String, PickupLocation>(props);
        // new topic rides_loaction
        var record = kafkaProducer.send(new ProducerRecord<>("rides_location", String.valueOf(pickupLocation.PULocationID), pickupLocation), (metadata, exception) -> {
            if (exception != null) {
                System.out.println(exception.getMessage());
            }
        });
        System.out.println(record.get().offset());
    }


    public static void main(String[] args) throws IOException, CsvException, ExecutionException, InterruptedException {
        var producer = new JsonProducerPickupLocation();
        producer.publish(new PickupLocation(186, LocalDateTime.now()));
    }
}
```

Create a new class called JsonKStreamJoins with the same Streams configuration as our JsonKStream class.

Here are our imports:

```java
package org.example;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.Topology;
import org.apache.kafka.streams.errors.StreamsUncaughtExceptionHandler;
import org.apache.kafka.streams.kstream.*;
import org.example.customserdes.CustomSerdes;
import org.example.data.PickupLocation;
import org.example.data.Ride;
import org.example.data.VendorInfo;

import java.time.Duration;
import java.util.Optional;
import java.util.Properties;
```

Here is the skeleton of our class with the application ID updated:

```java
public class JsonKStreamJoins {

	private Properties props = new Properties();

	public JsonKStreamJoins() {
		String userName = System.getenv("CLUSTER_API_KEY");
		String passWord = System.getenv("CLUSTER_API_SECRET");
		String bootstrapServer = System.getenv("BOOTSTRAP_SERVER");
		props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServer);
		props.put("security.protocol", "SASL_SSL");
		props.put("sasl.jaas.config", String.format("org.apache.kafka.common.security.plain.PlainLoginModule required username='%s' password='%s';", userName, passWord));
		props.put("sasl.mechanism", "PLAIN");
		props.put("client.dns.lookup", "use_all_dns_ips");
		props.put("session.timeout.ms", "45000");
		// update application id
		props.put(StreamsConfig.APPLICATION_ID_CONFIG, "kafka_tutorial.kstream.joined.rides.pickuplocation.v1");
		props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest");
props.put(StreamsConfig.CACHE_MAX_BYTES_BUFFERING_CONFIG, 0);
	}
	
	public static void main(String[] args) {
		var object = new JsonKStreamJoins();
	}
}
```

Then we will create two methods and include some inputs:

```java

private static final String INPUT_RIDE_TOPIC = "rides";
private static final String INPUT_RIDE_LOCATION_TOPIC = "rides_location";
private static final String OUTPUT_TOPIC = "vendor_info";

// next video will talk about topology
public Topology createTopology() {

}

public void joinRidesPickupLocation() throws InterruptedException {

	var topology = createTopology();
	var kStreams = new KafkaStreams(topology, props);
	// catching exceptions
	kStreams.setUncaughtExceptionHandler(exception -> {
	System.out.println(exception.getMessage());
	return StreamsUncaughtExceptionHandler.StreamThreadExceptionResponse.SHUTDOWN_APPLICATION;
	});
	//starting the stream
	kStreams.start();
	
	while (kStreams.start() != KafkaStreams.State.RUNNING) {
	System.out.println(kStreams.state());
	Thread.sleep(1000);
	}
	
System.out.println(kStreams.state());

Runtime.getRuntime().addShutdownHook(new Thread(kStreams::close));

}
```
Now let's build a topology.

```java
public Topology createTopology() {
	// need streambuilder and data from input topic(s)
	StreamsBuilder streamsBuilder = new StreamsBuilder();
	KStream<String, Ride> rides = streamsBuilder.stream(INPUT_RIDE_TOPIC, Consumed.with(Serdes.String(), CustomSerdes.getSerde(Ride.class)));
	KStream<String, PickupLocation> pickupLocations = streamsBuilder.stream(INPUT_RIDE_LOCATION_TOPIC, Consumed.with(Serdes.String(), CustomSerdes.getSerde(PickupLocation.class)));
	
	var pickupLocationsKeyedOnPUId = pickupLocations.selectKey((key, value) -> String.valueOf(value.PULocationID));
	var joined = rides.join(pickupLocationsKeyedOnPUId, (ValueJoiner<Ride, PickupLocation, Optional<VendorInfo>>) (ride, pickupLocation) -> {
		// time elapsed between calls
		var period = Duration.between(ride.tpep_dropoff_datetime, pickupLocation.tpep_pickup_datetime);
		// Optional is wrapper around null
		if(period.abs().toMinutes() > 10) return Optional.empty();
		else return Optional.of(new VendorInfo(ride.VendorID, pickupLocation.PULocationID, pickupLocation.tpep_pickup_datetime, ride.tpep_dropoff_datetime));
	},
		JoinWindows.ofTimeDifferenceAndGrace(Duration.ofMinutes(20), Duration.ofMinutes(5)),
	StreamJoined.with(Serdes.String(), CustomSerdes.getSerde(Ride.class), CustomSerdes.getSerde(PickupLocation.class)));

	joined.filter(((key, value) -> value.isPresent())).mapValues(Optional::get).to(OUTPUT_TOPIC, Produced.with(Serdes.String(), CustomSerdes.getSerde(VendorInfo.class)));

	// returns the topology
	return streamsBuilder.build();
}
```

Then make sure our main class throws InterruptedException (from joinRidesPickupLocation method).

Now when we run the two producers and this new stream we have some messages being produced through out new kafka_tutorial_kstream.joined.rides.pickuplocation.v1 topic.![[Screenshot 2024-03-06 at 8.44.45 AM.png]]

# Kafka Stream Testing

We have already created a basic Kafka stream example and so now we are going to write unit tests for it. In these examples, we used two classes from Kafka streams: Stream builder and KStreams. In the Stream builder, this is where we tell them which topics to read from, what are the actions on the events we want to do, and where to output and this is called a **topology**. We can test the **topology** with something called a topology driver. To do this, we need to write a function that will return the topology and then test it.

Going back to our count example (JsonKSream.java), we need to write our code in a way so that the topology can be extracted and then test it.

First let's update the import list to include CustomSerdes:

```java
import org.example.customserdes.CustomSerdes;
import java.util.Optional;
```

We will also update the initialization of the class so that it can take in existing properties:

```java
public JsonKStream(Optional<Properties> properties) {

	this.props = properties.orElseGet(() -> {
	
		String userName = System.getenv("CLUSTER_API_KEY");
		String passWord = System.getenv("CLUSTER_API_SECRET");
		String bootstrapServer = System.getenv("BOOTSTRAP_SERVER");
		props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServer);
		props.put("security.protocol", "SASL_SSL");
		props.put("sasl.jaas.config", String.format("org.apache.kafka.common.security.plain.PlainLoginModule required username='%s' password='%s';", userName, passWord));
		props.put("sasl.mechanism", "PLAIN");
		props.put("client.dns.lookup", "use_all_dns_ips");
		props.put("session.timeout.ms", "45000");
		props.put(StreamsConfig.APPLICATION_ID_CONFIG, "kafka_tutorial.kstream.count.plocation.v1");
		props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest");
props.put(StreamsConfig.CACHE_MAX_BYTES_BUFFERING_CONFIG, 0);
		
		return props;
	});
}
```

Then in our main class when we call JsonKStream we need it feed it an optional empty argument:
```java
var object = new JsonKStream(Optional.empty());
```

Now we will make a new class called createTopology which will return a Topology and just has the code in the countPLocation class which sets up the operations we want to do on the topics.

```java
public Topology createTopology() {
	StreamsBuilder streamsBuilder = new StreamsBuilder();
	// returns a kafka stream
	var ridesStream = streamsBuilder.stream("rides", Consumed.with(Serdes.String(), CustomSerdes.getSerde(Ride.class)));
	var puLocationCount = ridesStream.groupByKey().count().toStream();
	puLocationCount.to("rides-pulocation-count", Produced.with(Serdes.String(), Serdes.Long()));
	// return topology
	return streamsBuilder.build();
}
```

Then we can update our countPLocation class to create the topology by just calling this class:

```java
public void countPLocation() {
	var topology = createTopology();
	var kStreams = new KafkaStreams(topology, props);
	kStreams.start()

	Runtime.getRuntime().addShutdownHook(new Thread(kStreams::close));
}
```

Alright so now we have isolated the creation of our topology for testing. Let's create a new set of folders in the src directory: test/java/org/example. And within that we will have a new file called JsonKStreamsTest.java with this skeleton populated:

```java
// imports
package org.example;

import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.*;
import org.example.customserdes.CustomSerdes;
import org.example.data.Ride;
import org.example.helper.DataGeneratorHelper;
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;
import java.util.Properties;

class JsonKStreamTest {
private Properties props;
private static TopologyTestDriver testDriver;
private TestInputTopic<String, Ride> inputTopic;
private TestOutputTopic<String, Long> outputTopic;
private Topology topology;

@BeforeEach
public void setup() {
	props = new Properties();
	props.setProperty(StreamsConfig.APPLICATION_ID_CONFIG, "testing_count_application");
	props.setProperty(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "dummy:1234");
	if testDriver != null {
		testDriver.close();
	}

}

@Test
public void testIfOneMessageIsPassedToInputTopicWeGetCountOfOne() {

}

@AfterAll
public static void tearDown() { testDriver.close(); }
	if (testDriver != null) {
		testDriver.close();
	}
}

```

Then we can create our topology in this setup after we set the properties:

```java
topology = new JsonKStream(Optional.of(props)).createTopology();
```

And create our input and output topics within the setup class:

```java
testDriver = newTopologyTestDriver(topology, props);
	inputTopic = testDriver.createInputTopoic("rides", Serdes.String().serializer(), CustomSerdes.getSerdes(Rides.class).serializer());
	ouptputTopic = testDriver.createOutputTopic("rides-pulocation-count", Serdes.String().deserializer(), Serdes.Long().deserializer());
```

Now we will create a folder in the current directory called helper with a file called DataGeneratorHelper.java the following helper class which will generate one example event from each topic:

```java
package org.example.helper;

import org.example.data.PickupLocation;
import org.example.data.Ride;
import org.example.data.VendorInfo;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.List;

public class DataGeneratorHelper {
    public static Ride generateRide() {
        var arrivalTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        var departureTime = LocalDateTime.now().minusMinutes(30).format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        return new Ride(new String[]{"1", departureTime, arrivalTime,"1","1.50","1","N","238","75","2","8","0.5","0.5","0","0","0.3","9.3","0"});
    }

    public static PickupLocation generatePickUpLocation(long pickupLocationId) {
        return new PickupLocation(pickupLocationId, LocalDateTime.now());
    }
}
```

Then back in our test class, we can have the following test:

```java
@Test
public void testIfOneMessageIsPassedToInputTopicWeGetCountOfOne() {
	Ride ride = DataGeneratorHelper.generateRide();
	inputTopic.pipeInput(String.valueOf(ride.DOLocationID), ride);

	assertEquals(outputTopic.getQueueSize(), 1);
	// L is for long type here
	assertEquals(outputTopic.readKeyValue(), KeyValue.pair(String.valueOf(ride.DOLocationID), 1L));
	assertTrue(outputTopic.isEmpty());
}
```

When we run the test, it passes!

Let's create another test that does counting for two events with two different keys:

```java
@Test
public void testIfTwoMessagesArePassedWithDifferentKey() {

	Ride ride1 = DataGeneratorHelper.generateRide();
	ride1.DOLocationID = 1L;
	inputTopic.pipeInput(String.valueOf(ride1.DOLocationID), ride1);

	Ride ride2 = DataGeneratorHelper.generateRide();
	ride2.DOLocationID = 200L;
	inputTopic.pipeInput(String.valueOf(ride2.DOLocationID, ride2));

	assertEquals(outputTopic.readKeyValue(), KeyValue.pair("1", 1L));
	assertEquals(outputTopic.readKeyValue(), KeyValue.pair("200", 1L));
	assertTrue(outputTopic.isEmpty());

}
```

And another that does counting when two events come in with the same key:

```java
@Test
public void testIfTwoMessagesArePassedWithSameKey() {

	Ride ride1 = DataGeneratorHelper.generateRide();
	ride1.DOLocationID = 100L;
	inputTopic.pipeInput(String.valueOf(ride1.DOLocationID), ride1);

	Ride ride2 = DataGeneratorHelper.generateRide();
	ride2.DOLocationID = 100L;
	inputTopic.pipeInput(String.valueOf(ride2.DOLocationID, ride2));

	assertEquals(outputTopic.readKeyValue(), KeyValue.pair("100", 1L));
	assertEquals(outputTopic.readKeyValue(), KeyValue.pair("100", 2L));
	assertTrue(outputTopic.isEmpty());

}
```

Alright! Now we are going to create a test for our JsonKStreamsJoin class. We will need to update the imports of that file as well as taking in the optional properties as we did for JsonKStream class.

```java
//import updates
import java.util.Optional;
```

```java
// udpate to take in optional props
public JsonKStreamJoins(Optional<Properties> properties) {

	this.props = properties.orElseGet(() -> {
	
		String userName = System.getenv("CLUSTER_API_KEY");
		String passWord = System.getenv("CLUSTER_API_SECRET");
		String bootstrapServer = System.getenv("BOOTSTRAP_SERVER");
		props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServer);
		props.put("security.protocol", "SASL_SSL");
		props.put("sasl.jaas.config", String.format("org.apache.kafka.common.security.plain.PlainLoginModule required username='%s' password='%s';", userName, passWord));
		props.put("sasl.mechanism", "PLAIN");
		props.put("client.dns.lookup", "use_all_dns_ips");
		props.put("session.timeout.ms", "45000");
		// update application id
		props.put(StreamsConfig.APPLICATION_ID_CONFIG, "kafka_tutorial.kstream.joined.rides.pickuplocation.v1");
props.put(StreamsConfig.CACHE_MAX_BYTES_BUFFERING_CONFIG, 0);
		
		return props;
	
	});

}
```

Then don't forget to update the main method with:

```java
var object = new JsonKStreamJoins(Optional.empty());
```

Now in this file we already separated the creation of the topology into a method. So now we will move on to created the test class called JsonKStreamJoinsTest.java:

Imports first:

```java
package org.example;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.internals.Topic;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.*;
import org.example.customserdes.CustomSerdes;
import org.example.data.PickupLocation;
import org.example.data.Ride;
import org.example.data.VendorInfo;
import org.example.helper.DataGeneratorHelper;
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import javax.xml.crypto.Data;
import java.util.Properties;
import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;
```

Now initialize the class:

```java
class JsonKStreamJoinsTest {

	private Properties props;
	private static TopologyTestDriver testDriver;
	private TestInputTopic<String, Ride> ridesTopic;
	private TestInputTopic<String, PickupLocation> pickupLocationTopic;
	private TestOutputTopic<String, VendorInfo> outputTopic;
	private Topology topology;

	@BeforeEach
	public void setup() {
		props = new Properties();
		props.setProperty(StreamsConfig.APPLICATION_ID_CONFIG, "testing_count_application");
		props.setProperty(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "dummy:1234");
		topology = new JsonKStreamJoins(Optional.of(props)).createTopology();
		if (testDriver != null) {
			testDriver.close();
		}
	
		testDriver = new TopologyTestDriver(topology, props);
		
		ridesTopic = testDriver.createInputTopic("rides", Serdes.String().serializer(), CustomSerdes.getSerde(Ride.class).serializer());
	
		pickupLocationTopic = testDriver.createInputTopic("rides_location", Serdes.String().serializer(), CustomSerdes.getSerde(PickupLocation.class).serializer());
		
		outputTopic = testDriver.createOutputTopic("vendor_info", Serdes.String().deserializer(), CustomSerdes.getSerde(VendorInfo.class).deserializer());
	}

	@AfterAll
	public static void shutdown() {
		if(testDriver != null) {
			testDriver.close();
		}
	}
}
```

Now let's create a test!

```java
@Test
public void testIfJoinWorksOnSameDropOffPickupLocationId() {
	Ride ride = DataGeneratorHelper.generateRide();

	PickupLocation pickupLocation = DataGeneratorHelper.generatePickUpLocation(ride.DOLocationID);
	ridesTopic.pipeInput(String.valueOf(ride.DOLocationID), ride);
	pickupLocationTopic.pipeInput(String.valueOf(pickupLocation.PULocationID), pickupLocation);

	assertEquals(outputTopic.getQueueSize(), 1);
	var expected = new VendorInfo(ride.VendorID, pickupLocation.PULocationID, pickupLocation.tpep_pickup_datetime, ride.tpep_dropoff_datetime);
	var result = outputTopic.readKeyValue();
	assertEquals(result.key, String.valueOf(ride.DOLocationID));
	assertEquals(result.value.VendorID, expected.VendorID);
	assertEquals(result.value.pickupTime, expected.pickupTime);

}
```

And the test passes!

# Kafka Stream Windowing

We will first talk about what a global Ktable is, but you can think of it as being similar to broadcasting. Let's say we have two nodes of the Kafka stream application running and both are reading and creating a KTable internally. The KTable is partitioned based upon the topic. So if the topic has 2 partitions, the KTable will also have two partitions with each part being given to a distinct node. So the nodes only have partial data because of this partitioning. Often this requires reshuffling, but reshuffling is costly and so that's what a Global KTable is trying to avoid. Okay so let's assume that each node has instead of a KTable, a Complete Global KTable which means the complete data is available to each node. This avoids reshuffling, but because the whole table is stored on the node itself there can be memory issues. So this Global KTable is best when you have a smaller table. If the data size is too big, you can't use a Global KTable. How we build this Global KTable is very simple:

```java
streamBuilder.globaltable("topic_name");
```

Now let's talk about different join types. Kafka Streams supports three kinds of joins:
* inner
	* KStream - KStream
	* KTable - KTable
	* KStream - KTable
	* KStream - GlobalKTable
* left
	*  KStream - KStream
	* KTable - KTable
	* KStream - KTable
	* KStream - GlobalKTabl
* outer
	* KStream - KStream
	* KTable - KTable

With an inner stream-stream join between views and clicks on a 10-second interval, let's say the Views stream looks like this:
0: A
1: B
2:
3: C
4: D
5:
6: F1 F2
7:
8: G
9: 
10: 
11: 
and our Clicks stream looks like this:
0: 
1: A
2: C
3: 
4: 
5: E
6: 
7: F
8: 
9: G1 G2
10: 
(just after) 11: B < - not in 10-second window
then the inner join result is the following:
0:
1: (A, A)
2:
3: (C, C)
4: 
5: 
6: 
7: (F1, F) (F2, F)
8: 
9: (G, G1) (G, G2)
10: 
11: 

Okay let's talk more about KStream windowing:
* Tumbling: fixed size non overlapping
	* For example:![[Screenshot 2024-03-08 at 12.53.38 PM.png]]
* Hopping: fixed size and overlapping
	* For example:
		  ![[Screenshot 2024-03-08 at 12.55.46 PM.png]]
* Sliding: fixed-size overlapping windows that work on differences between record and timestamps
* Session: dynamically-sized, non-overlapping, data-driven windows
	* Example:
		  ![[Screenshot 2024-03-08 at 12.57.02 PM.png]]

Let's see what this looks like in code. Create a JsonKStreamWindow class by making a copy of our JsonKStream class. Be sure to update the class name and the initializing method accordingly and also the call of the class in the main function. We will adjust the countPLocation class by renaming it countPLocationWindowed.

Add these imports:

```java
import org.apache.kafka.streams.kstream.TimeWindows;
import org.apache.kafka.streams.kstream.WindowedSerdes;
import java.time.Duration;
import java.time.temporal.ChronoUnit;
```

First let's take a look at our topology. Instead of counting everything for puLocationCount, we want to just count things in chunks (for now let's doing tumbling). 

```java
public Topology createTopology() {

	StreamsBuilder streamsBuilder = new StreamsBuilder();
	var ridesStream = streamsBuilder.stream("rides", Consumed.with(Serdes.String(), CustomSerdes.getSerde(Ride.class)));
	var puLocationCount = ridesStream.groupByKey().windowedBy(TimeWindows.ofSizeAndGrace(Duration.ofSeconds(10), Duration.ofSeconds(5))).count().toStream();
	var windowSerde = WindowedSerdes.timeWindowedSerdeFrom(String.class, 10*1000);
	puLocationCount.to("rides-pulocation-window-count", Produced.with(windowSerde, Serdes.Long()));
	
	return streamsBuilder.build();

}
```

And then create the "rides-pulocation-window-count" topic in Confluent Cloud before running this.

Also don't forget to start your JsonProducer, or else no events will be coming in!

# Kafka ksqldb & Connect

ksqldb is Kafka's way of having SQL to do really quick analysis with respect to your streams coming in. It can be used for analytical/testing processing or even in production (but be careful).

In Confluent Cloud, navigate to our data-engineering-zoomcamp-cluster cluster and you will see ksqlDB as an option in the menu on the left. Then you can select the button "create  cluster myself" and choose global access with the default name.
![[Screenshot 2024-03-10 at 7.15.49 PM.png]]

We will focus on the rides topic for now. Once your cluster is done provisioning, in the editor we will have the following kSQL statement:

```SQL
CREATE STREAM ride_streams (
	VendorID varchar,
	trip_distance double,
	payment_type varchar,
	passenger_count double
) WITH (KAFKA_TOPIC='rides',
		VALUE_FORMAT='json');
```
Update the auto.offset.reset to Earliest so we get all the event data available:
![[Screenshot 2024-03-10 at 7.23.30 PM.png]]
Now since our data was deleted every 24 hours, we may need to run our JSONProducer again to get data going through the stream.

When we run the query in ksqlDB we get the following output:

![[Screenshot 2024-03-10 at 7.25.09 PM.png]]

Now we can see the RIDE_STREAMS in the Flow tab and it is only generating the data we specifically queried:
![[Screenshot 2024-03-10 at 7.26.35 PM.png]]

Now if we go back to the editor we can query our new stream:

```SQL
SELECT *
FROM ride_streams EMIT CHANGES;
```

And we can see the data that has been generated thus far through our new stream:
![[Screenshot 2024-03-10 at 7.29.07 PM.png]]

We can try some other queries:

```SQL
SELECT COUNT(*)
FROM ride_streams
EMIT CHANGES;
```

![[Screenshot 2024-03-10 at 7.30.54 PM.png]]

And yet another example:

```SQL
SELECT
	payment_type,
	COUNT(*)
FROM ride_streams
GROUP BY payment_type
EMIT CHANGES;
```
![[Screenshot 2024-03-10 at 7.32.56 PM.png]]

You can also filter (using the WHERE clause). Now the EMIT CHANGES option will cause for more outputs to be generated if data is being streamed in.

We can create tables and also indicate a window of time where new output will be generated depending on the indicated session:

```SQL
CREATE TABLE payment_type_sessions AS
	SELECT payment_type,
		COUNT(*)
	FROM ride_streams
	WINDOW SESSION (60 SECONDS)
	GROUP BY payment_type
	EMIT CHANGES;
```

Since this table will be updated every 60 seconds, it is a persistent query which means it is running continuously in the background. You can delete persistent queries such as this one using the Persistent queries tab.

Note that there is a Java client that you can wrap around ksqlDB. The disadvantage to using ksqlDB is that you have to have a separate cluster altogether from what is used to generate the streaming data. But it is good for proof-of-concepts or prototyping, otherwise you should use Kafka Streams.

Now let's take a look at Connectors:

![[Screenshot 2024-03-10 at 7.44.19 PM.png]]

Kafka Connect allows you to connect to a variety of sources/sinks where you pull data from or put data into. For example, if we use Elasticsearch Service Sink connector we can select the topics we want to export and it will ask for the API key file / connection URI, etc.

# Kafka Schema registry

Here we will talk about a few special scenarios, for example: What happens if the producers and consumers are not speaking the same language? What if the producers change the format? In the case of streaming, this can happen if you change the type of the message you are producing (e.g., Json with VendorID as string, then change it to Json with VendorID as integer). Schemas prevent us from making mistakes when we have changes we want to make. Schemas are a contract between producers and consumers. Producers generate the schema and then the schema is distributed to the consumer. When producers and consumers have compatible schemas, they can talk to each other. You can have multiple schemas (those are, for example, changes to messages being produced) but they have to be compatible with each other. The **schema registry** is the one that takes care of the compatibility of schemas.

The producer first publishes its schema to the schema registry. The schema registry gives an acknowledgment to mean "go ahead and produce". The schema registry can tell the producer no if the proposed schema is not compatible with the existing schema. In this case, the producer will not be able to produce. The consumers read the schema from the schema registry.

We will do an example where we switch from Json to Avro. **Avro** is a data serialization format that is open source and was generated for big data technologies to make producers and consumers compatible. The schema is dictionary format like Json but the data itself is in binary so that it is very efficient compared to Json.

There are lots of benefits for using Avro with respect to Kafka:

![[Screenshot 2024-03-10 at 8.06.52 PM.png]]

With respect to compatibilities, we have to think of three different kinds:
* Forward compatibility: producers can write with an updated version and consumers can read with the previous version (for example, adding an optional field)
* Backward compatibility: producers can write with the previous version and consumers can read with the new version 
* Full compatibility: any producer can produce from any version and consumers can consume from any different version

In our main folder within the Java project, let's create a subfolder called **avro** with three files:
* rides.avsc
* rides_compatible.avsc
* rides_non_compatible.avsc
Our rides.avsc file will contain the following:

```avro
{
	"type": "record",
	"name": "RideRecord",
	"namespace": "schemaregistry",
	"fields": [
		{"name":"vendor_id", "type":"string"},
		{"name":"passenger_count", "type":"int"},
		{"name":"trip_distance", "type":"double"},
	]
}
```

Our rides_compatible.avsc file will contain the following:

```avro
{
   "type": "record",
       "name":"RideRecordCompatible",
       "namespace": "schemaregistry",
       "fields":[
         {"name":"vendorId","type":"string"},
         {"name":"passenger_count","type":"int"},
         {"name":"trip_distance","type":"double"},
         {"name":"pu_location_id", "type": [ "null", "long" ], "default": null}
       ]
}
```

Our rides_non_compatible.avsc file will contain:

```avro
{
   "type": "record",
       "name":"RideRecordNoneCompatible",
       "namespace": "schemaregistry",
       "fields":[
         {"name":"vendorId","type":"int"},
         {"name":"passenger_count","type":"int"},
         {"name":"trip_distance","type":"double"}
       ]
}
```


In our gradle file, there is an avro plugin we are using. Be sure to also have the gradle brew installed, make the gradle wrapper available locally:

```bash
brew install gradle
gradle wrapper
```

have the gradlew, gradlew.bat, and settings.gradle files and then run the following commands:

```bash
chmod +x gradlew
./gradlew clean
./gradelw build
```

We can see in our build directory there is now a folder generated-main-avro-java containing files generated based on our schema registry. One is, for example RideRecord from our rides_avro.avsc file. This is created by the plugin.

Now we need to update our getRides method so that it outputs a list of objects of type RideRecord instead of just Ride.

We will start with this. Go to Confluent Cloud and create a new topic called rides_avro (with 2 partitions and retention 1 day).

Now make a copy of the JsonProducer, rename it AvroProducer. Add these imports:

```java
import io.confluent.kafka.serializers.AbstractKafkaAvroSerDeConfig;
import io.confluent.kafka.serializers.KafkaAvroSerializer;
import schemaregistry.RideRecord;
```
and change the value serializer to the following:

```java
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, KafkaAvroSerializer.class.getName());
```

We now have to add some information about our schema registry. To get a key for our Schema Registry, go to Environment on the left, click on Default, and then look in the lower right corner. You will see a section where you can add a key under Credentials in the Stream Governance API section:
![[Screenshot 2024-03-10 at 8.34.58 PM.png]]

Go ahead and create the key and export the key and secret as environment variables (the first is the Endpoint URL we see here):
SCHEMA_REGISTRY_URL
SCHEMA_REGISTRY_KEY
SCHEMA_REGISTRY_SECRET

Then we add the following to our class initializer:

```java
String schemaRegUrlConfig = System.getenv("SCHEMA_REGISTRY_URL");
String schemaRegUserName = System.getenv("SCHEMA_REGISTRY_KEY");
String schemaRegPassWord = System.getenv("SCHEMA_REGISTRY_SECRET");

props.put(AbstractKafkaAvroSerDeConfig.SCHEMA_REGISTRY_URL_CONFIG, schemaRegUrlConfig);
props.put("basic.auth.credentials.source", "USER_INFO");
props.put("basic.auth.user.info",schemaRegUserName+":"+schemaRegPassWord);
```

Now we are updating our methods as follows:

```java
public List<RideRecord> getRides() throws IOException, CsvException {
	var ridesStream = this.getClass().getResource("/rides.csv");
	var reader = new CSVReader(new FileReader(ridesStream.getFile()));
	reader.skip(1);
	return reader.readAll().stream().map(row ->
		RideRecord.newBuilder().setVendorId(row[0])
		.setTripDistance(Double.parseDouble(row[4]))
		.setPassengerCount(Integer.parseInt(row[3]))
		.build()
		).collect(Collectors.toList());
}

  

public void publishRides(List<RideRecord> rides) throws ExecutionException, InterruptedException {
	KafkaProducer<String, RideRecord> kafkaProducer = new KafkaProducer<String, RideRecord>(props);
	for(RideRecord ride: rides) {
		var record = kafkaProducer.send(new ProducerRecord<>("rides_avro", String.valueOf(ride.getVendorId()), ride), (metadata, exception) -> {
		if(exception != null) {
		System.out.println(exception.getMessage());
		}
		});
		Thread.sleep(500);
	}
}

public static void main(String[] args) throws IOException, CsvException, ExecutionException, InterruptedException {
	var producer = new AvroProducer();
	var rideRecords = producer.getRides();
	producer.publishRides(rideRecords);
}
```

Get the JsonKStream going and this file so that we can see events populating in our rides_avro topic:


![[Screenshot 2024-03-10 at 9.29.51 PM.png]]

We can see our schema on the Schema tab:

![[Screenshot 2024-03-10 at 9.33.30 PM.png]]

While we are here, change the compatibility mode to Transitive full:
![[Screenshot 2024-03-10 at 9.34.35 PM.png]]

Now notice our rides_non_compatible schema has vendorID as type int instead of string which makes the schema non-compatible to the previous version. Change the name to RideRecord, change the name of rides.avsc to RideRecordPrevious, update this bit in AvroProducer:

```java
	.setVendorId(Integer.parseInt(row[0]))
```
clean and build the gradle, run this producer and this is the error we are receiving:

```
Schema being registered is incompatible with an earlier schema for subject "rides_avro-value", details: [{errorType:'READER_FIELD_MISSING_DEFAULT_VALUE', description:'The field 'vendor_id' at path '/fields/0' in the old schema has no default value and is missing in the new schema', additionalInfo:'vendor_id'}
```
Now if you look at rides_compatible.avsc the types are the same but there is an optional field added. Change the name to RideRecord (keep the rides.avsc name to RideRecordPrevious, revert the rides_non_compatible name to RideRecordNonCompatible). Revert the VendorId changes back, add the following to the getRides method in our AvroProducer:

```java
.setPuLocationId(Long.parseLong(row[7]))
```

clean and build the cradle, run AvroProducer and then we can see our schema updated:

![[Screenshot 2024-03-10 at 10.00.12 PM.png]]

# Streaming with PySpark

We will be following much of what was done in the Java Steaming videos, but instead now in Python. However, instead of using Confluent Cloud we will use a Docker container that has the Kafka services setup within it.

Once you have all the files in the docker folder provided in the repo, navigate to the docker folder and run the following commands:

```bash
chmod +x build.sh
./build.sh
```

Now we will create a network and volume for our containers:

```bash
# Create Network
docker network  create kafka-spark-network

# Create Volume
docker volume create --name=hadoop-distributed-file-system
```

Then check to make sure these are both running:
```bash
docker volume ls # should list hadoop-distributed-file-system
docker network ls # should list kafka-spark-network 
```

Now for this first part, we only need the Kafka services up and running. So navigate to the Kafka folder and run:

```bash
docker compose up -d
```

In our Kafka docker-compose.yml file, the broker service specifies the KAFKA_LISTENERS and the KAFKA_ADVERTISED_LISTENERS. These two parameters say how Kafka communicates within Docker and how we can access the broker outside Docker. In our file, it says we can access it at localhost:9092.

In your virtual environment, make sure to have python-kafka pip installed. Then go to the codec.py file in the kafka folder of your virtual environment and change

```python
from kafka.vendor.six.moves import range
```

to

```python
from six.moves import range
```

We also want to pip install confluent_kafka, requests, and fastavro.

Let's start now with the JsonProducer producer.py. The aim is to read a csv file and create a producer that references a csv file on our local machine and publish each row into a specific Kafka topic. Inside our pyspark_streaming_examples folder, make sure rides.csv is uploaded to a subfolder called resources. We will create a subfolder within the pyspark_streaming_examples directory called json_example. Within that is a settings.py file containing the following:

```python
INPUT_DATA_PATH = "../resources/rides.csv"

BOOTSTRAP_SERVICES = ["localhost:9092"]
KAFKA_TOPIC = "rides_json"
```

Now we will create a ride.py file in this json_example defining the class Ride with the following content:

```python
from typing import List, Dict
from decimal import Decimal
from datetime import datetime


class Ride:
    def __init__(self, arr: List[str]):
        self.vendor_id = arr[0]
        self.tpep_pickup_datetime = datetime.strptime(arr[1], "%Y-%m-%d %H:%M:%S"),
        self.tpep_dropoff_datetime = datetime.strptime(arr[2], "%Y-%m-%d %H:%M:%S"),
        self.passenger_count = int(arr[3])
        self.trip_distance = Decimal(arr[4])
        self.rate_code_id = int(arr[5])
        self.store_and_fwd_flag = arr[6]
        self.pu_location_id = int(arr[7])
        self.do_location_id = int(arr[8])
        self.payment_type = arr[9]
        self.fare_amount = Decimal(arr[10])
        self.extra = Decimal(arr[11])
        self.mta_tax = Decimal(arr[12])
        self.tip_amount = Decimal(arr[13])
        self.tolls_amount = Decimal(arr[14])
        self.improvement_surcharge = Decimal(arr[15])
        self.total_amount = Decimal(arr[16])
        self.congestion_surcharge = Decimal(arr[17])

    @classmethod
    def from_dict(cls, d: Dict):
        return cls(arr=[
            d['vendor_id'],
            d['tpep_pickup_datetime'][0],
            d['tpep_dropoff_datetime'][0],
            d['passenger_count'],
            d['trip_distance'],
            d['rate_code_id'],
            d['store_and_fwd_flag'],
            d['pu_location_id'],
            d['do_location_id'],
            d['payment_type'],
            d['fare_amount'],
            d['extra'],
            d['mta_tax'],
            d['tip_amount'],
            d['tolls_amount'],
            d['improvement_surcharge'],
            d['total_amount'],
            d['congestion_surcharge'],
        ]
        )

    def __repr__(self):
        return f'{self.__class__.__name__}: {self.__dict__}'
```

Then we will create another file in this json_example folder called producer.py with the following content:

* Imports

```python
import csv
import json
from typing import List, Dict
from kafka import KafkaProducer
from kafka.errors import KafkaTimeoutError

from ride import Ride
from settings import BOOTSTRAP_SERVICES, INPUT_DATA_PATH, KAFKA_TOPIC
```

Then we create our JsonProducer class:

```python
class JsonProducer(KafkaProducer):
	def __init__(self, props:Dict):
		self.producer = KafkaProducer(**props)

	@staticmethod
	def read_records(resource_path: str):
		records = []
		with open(resource_path, 'r') as f:
			reader = csv.reader(f)
			header = next(reader) # skip the header row
			for row in reader:
				records.append(Ride(arr=row))
		return records

	def publish_rides(self, topic: str, messages: List[Ride]):
		for ride in messages:
			try:
				record = self.producer.send(topic=topic, key = ride.pu_location_id, value=ride)
				print('Record {} successfully produced at offset {}'.format(ride.pu_location_id, record.get().offset)
			except KafkaTimeoutError as e:
				print(e.__str__())

```

Notice in the configuration below, the key is an integer so we convert it into a string before encoding it. Similarly, we convert x into a dictionary and then json.dumps converts it into a string before encoding.
```python
if __name__ == '__main__':
	# Config should match with the KafkaProducer expectation
	config = {
		'bootstrap_servers': BOOTSTRAP_SERVERS,
		'key_serializer': lambda key: str(key).encode(),
		'value_serializer': lambda x: json.dumps(x.__dict__, default=str).encode('utf-8')
	}
	producer = JsonProducer(props=config)
	rides = producer.read_records(resource_path=INPUT_DATA_PATH)
	producer.publish_rides(topic=KAFKA_TOPIC, messages=rides)
```

Now we can run this file using:

```bash
python producer.py
```

and we see records are successfully be produced:

![[Screenshot 2024-03-11 at 8.26.35 AM.png]]

Now let's take a look at our consumer, create a consumer.py file with the following content:

Imports first:

```python
from typing import Dict, List
from json import loads
from kafka import KafkaConsumer

from ride import Ride
from settings import BOOTSTRAP_SERVERS, KAFKA_TOPIC
```

Then we initialize our JsonConsumer class:

```python
class JsonConsumer:

def __init__(self, props: Dict):
	self.consumer = KafkaConsumer(**props)
```

Within our class we have a consume_from_kafka method:

```python
def consume_from_kafka(self, topics: List[str]):
	self.consumer.subscribe(topics)
	print("Consuming from Kafka started")
	print("Available topics to consume: ", self.consumer.subscription())
	while True:
		try:
			# SIGINT can't be handled when polling, limit timeout to 1 second
			message = self.consumer.poll(1.0)
			if message is None or message == {}:
				continue
			for message_key, message_value in message.items():
				for msg_val in message_value:
					print(msg_val.key, msg_val.value)
		except KeyboardInterrupt:
			break
	
	self.consumer.close()
```

Then we have our main function:

```python
if __name__ == "__main__":
	config = {
		"bootstrap_servers": BOOTSTRAP_SERVERS,
		"auto_offset_reset": "earliest",
		"enable_auto_commit": True,
		"key_deserializer": lambda key: int(key.decode("utf-8")),
		"value_deserializer": lambda x: loads(
		x.decode("utf-8"), object_hook=lambda d: Ride.from_dict(d)
		),
		"group_id": "consumer.group.id.json-example.1",
	}
	
	json_consumer = JsonConsumer(props=config)
	json_consumer.consume_from_kafka(topics=[KAFKA_TOPIC])
```

Now we can run our consumer.py file and we will see the outputs:
![[Screenshot 2024-03-11 at 8.45.31 AM.png]]

Now let's move to the avro example. In our pyspark_streaming_examples folder, create a subfolder called avro_example. In it, we will have the settings.py file:

```python
INPUT_DATA_PATH = '../resources/rides.csv'

RIDE_KEY_SCHEMA_PATH = '../resources/schemas/taxi_ride_key.avsc'
RIDE_VALUE_SCHEMA_PATH = '../resources/schemas/taxi_ride_value.avsc'

SCHEMA_REGISTRY_URL = 'http://localhost:8081'
BOOTSTRAP_SERVERS = 'localhost:9092'
KAFKA_TOPIC = 'rides_avro'
```

We will have ride_record.py:

```python
from typing import List, Dict


class RideRecord:

    def __init__(self, arr: List[str]):
        self.vendor_id = int(arr[0])
        self.passenger_count = int(arr[1])
        self.trip_distance = float(arr[2])
        self.payment_type = int(arr[3])
        self.total_amount = float(arr[4])

    @classmethod
    def from_dict(cls, d: Dict):
        return cls(arr=[
            d['vendor_id'],
            d['passenger_count'],
            d['trip_distance'],
            d['payment_type'],
            d['total_amount']
        ]
        )

    def __repr__(self):
        return f'{self.__class__.__name__}: {self.__dict__}'


def dict_to_ride_record(obj, ctx):
    if obj is None:
        return None

    return RideRecord.from_dict(obj)


def ride_record_to_dict(ride_record: RideRecord, ctx):
    return ride_record.__dict__
```

Along with ride_record_key.py:

```python
from typing import Dict


class RideRecordKey:
    def __init__(self, vendor_id):
        self.vendor_id = vendor_id

    @classmethod
    def from_dict(cls, d: Dict):
        return cls(vendor_id=d['vendor_id'])

    def __repr__(self):
        return f'{self.__class__.__name__}: {self.__dict__}'


def dict_to_ride_record_key(obj, ctx):
    if obj is None:
        return None

    return RideRecordKey.from_dict(obj)


def ride_record_key_to_dict(ride_record_key: RideRecordKey, ctx):
    return ride_record_key.__dict__
```

Now in our resources folder, let's create a schemas subfolder with two files:
(1) taxi_ride_key.avsc
```avro
{
  "namespace": "com.datatalksclub.taxi",
  "type": "record",
  "name": "RideRecordKey",
  "fields": [
    {
      "name": "vendor_id",
      "type": "int"
    }
  ]
}
```
(2) taxi_ride_value.avsc
```avro
{
  "namespace": "com.datatalksclub.taxi",
  "type": "record",
  "name": "RideRecord",
  "fields": [
    {
      "name": "vendor_id",
      "type": "int"
    },
    {
      "name": "passenger_count",
      "type": "int"
    },
    {
      "name": "trip_distance",
      "type": "float"
    },
    {
      "name": "payment_type",
      "type": "int"
    },
    {
      "name": "total_amount",
      "type": "float"
    }
  ]
}
```

Okay so now let's take a look at our avro producer (producer.py file):

Imports first:
```python
import os
import csv
from time import sleep
from typing import Dict

from confluent_kafka import Producer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroSerializer
from confluent_kafka.serialization import SerializationContext, MessageField

from ride_record_key import RideRecordKey, ride_record_key_to_dict
from ride_record import RideRecord, ride_record_to_dict
from settings import RIDE_KEY_SCHEMA_PATH, RIDE_VALUE_SCHEMA_PATH, \
    SCHEMA_REGISTRY_URL, BOOTSTRAP_SERVERS, INPUT_DATA_PATH, KAFKA_TOPIC
```

When we define our RideAvroProducer class, we will make sure to load our schema:

```python

class RideAvroProducer:
	def __init__(self, props: Dict):
		# schema registry and serializer-deserializer configs
		key_schema_str = self.load_schema(props['schema.key'])
		value_schema_str = self.load_schema(props['schema.value'])
		schema_registry_props = {'url': props['schema_registry.url']}
		schema_registry_client = SchemaRegistryClient(schema_registry_props)
		self.key_serializer = AvroSerializer(schema_registry_client, key_schema_str, ride_record_key_to_dict)
		self.value_serializer = AvroSerializer(schema_registry_client, value_schema_str, ride_record_to_dict)

		# producer config
		producer_props = {'bootstrap.servers': props['bootstrap.servers']}
		self.producer = Producer(producer_props)
```

Now we will create a method that reads the schema definition from the specified place:

```python
@staticmethod
def load_schema(schema_path: str):
	path = os.path.realpath(os.path.dirname(__file__))
	with open(f"{path}/{schema_path}") as f:
		schema_str = f.read()
	return schema_str
```

Now our static read_records method looks like this:

```python
@staticmethod
def read_records(resource_path: str):
	ride_records, ride_keys = [], []
	with open(resource_path, 'r') as f:
		reader = csv.reader(f)
		header = next(reader) # skip the header
		for row in reader:
			ride_records.append(RideRecord(arr=[row[0], row[3], row[4], row[9], row[16]]))
			ride_keys.append(RideRecordKey(vendor_id=int(row[8])))
		return zip(ride_keys, ride_records)
```

We will have a delivery report method that sends a message to say either the message was delivered or not and there was an error:

```python
@staticmethod
def delivery_report(err, msg):
if err is not None:
	print("Delivery failed for record {}: {}".format(msg.key(), err))
	return

print('Record {} successfully produced to {} [{}] at offset {}'.format(
msg.key(), msg.topic(), msg.partition(), msg.offset()))
```

Then we publish our records:

```python
def publish(self, topic: str, records: [RideRecordKey, RideRecord]):
	for key_value in records:
		key, value = key_value
		try:
			self.producer.producer(topic=topic,
				key=self.key_serializer(key, SerializationContext(topic=topic,
					field=MessageField.KEY)),
				value = self.value_serializer(value, SerializationContext(topic=topic, field=MessageField.VALUE)),
				on_delivery = self.delivery_report)
		except KeyboardInterrupt:
			break
		except Exception as e:
			print(f"Exception while producing record - {value}: {e}")
			
	self.producer.flush()
	sleep(1)
			
```


Our main function specifies the schema registry:

```python
if __name__ == "__main__":
    config = {
        'bootstrap.servers': BOOTSTRAP_SERVERS,
        'schema_registry.url': SCHEMA_REGISTRY_URL,
        'schema.key': RIDE_KEY_SCHEMA_PATH,
        'schema.value': RIDE_VALUE_SCHEMA_PATH
    }
    producer = RideAvroProducer(props=config)
    ride_records = producer.read_records(resource_path=INPUT_DATA_PATH)
    producer.publish(topic=KAFKA_TOPIC, records=ride_records)
```

Now when we run this file, we see the messages are being sent, but remember that it is in binary format so that is why we are seeing something kind of strange here from our avro serialization:

![[Screenshot 2024-03-11 at 9.39.01 AM.png]]

Now let's move on to our consumer. Here are our imports:

```python
import os
from typing import Dict, List

from confluent_kafka import Consumer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroDeserializer
from confluent_kafka.serialization import SerializationContext, MessageField

from ride_record_key import dict_to_ride_record_key
from ride_record import dict_to_ride_record
from settings import BOOTSTRAP_SERVERS, SCHEMA_REGISTRY_URL, \
    RIDE_KEY_SCHEMA_PATH, RIDE_VALUE_SCHEMA_PATH, KAFKA_TOPIC
```

We then initialize our class similarly to before with the schema registry information:

```python
class RideAvroConsumer:
    def __init__(self, props: Dict):

        # Schema Registry and Serializer-Deserializer Configurations
        key_schema_str = self.load_schema(props['schema.key'])
        value_schema_str = self.load_schema(props['schema.value'])
        schema_registry_props = {'url': props['schema_registry.url']}
        schema_registry_client = SchemaRegistryClient(schema_registry_props)
        self.avro_key_deserializer = AvroDeserializer(schema_registry_client=schema_registry_client,
                                                      schema_str=key_schema_str,
                                                      from_dict=dict_to_ride_record_key)
        self.avro_value_deserializer = AvroDeserializer(schema_registry_client=schema_registry_client,
                                                        schema_str=value_schema_str,
                                                        from_dict=dict_to_ride_record)

        consumer_props = {'bootstrap.servers': props['bootstrap.servers'],
                          'group.id': 'datatalkclubs.taxirides.avro.consumer.2',
                          'auto.offset.reset': "earliest"}
        self.consumer = Consumer(consumer_props)
```

We load the schema as in producer:

```python
@staticmethod
    def load_schema(schema_path: str):
        path = os.path.realpath(os.path.dirname(__file__))
        with open(f"{path}/{schema_path}") as f:
            schema_str = f.read()
        return schema_str
```

And then we have a consume_from_kafka method that is very similar to the one for the json producer:

```python
def consume_from_kafka(self, topics: List[str]):
        self.consumer.subscribe(topics=topics)
        while True:
            try:
                # SIGINT can't be handled when polling, limit timeout to 1 second.
                msg = self.consumer.poll(1.0)
                if msg is None:
                    continue
                key = self.avro_key_deserializer(msg.key(), SerializationContext(msg.topic(), MessageField.KEY))
                record = self.avro_value_deserializer(msg.value(),
                                                      SerializationContext(msg.topic(), MessageField.VALUE))
                if record is not None:
                    print("{}, {}".format(key, record))
            except KeyboardInterrupt:
                break

        self.consumer.close()
```

Finally our main function:

```python
if __name__ == "__main__":
    config = {
        'bootstrap.servers': BOOTSTRAP_SERVERS,
        'schema_registry.url': SCHEMA_REGISTRY_URL,
        'schema.key': RIDE_KEY_SCHEMA_PATH,
        'schema.value': RIDE_VALUE_SCHEMA_PATH,
    }
    avro_consumer = RideAvroConsumer(props=config)
    avro_consumer.consume_from_kafka(topics=[KAFKA_TOPIC])
```

We see our messages are being consumed properly:
![[Screenshot 2024-03-11 at 9.43.07 AM.png]]

# PySpark Structured Streaming

In this section, we will see how to communicate between a Spark cluster and a Kafka cluster. 

In the Spark folder, run the build.sh file:
```bash
./build.sh
```

Make sure the SPARK_VERSION matches the spark version on your local computer using:

```bash
spark-submit --version
```

For me, it is 3.5.1.

Then get the docker container in the spark folder is up and running.

In our python_examples folder (renamed from pyspark_streaming_examples), create a folder called streams-example with a subfolder called pyspark.

In the pyspark folder, we will having the following settings.py file:

```python
import pyspark.sql.types as T

INPUT_DATA_PATH = '../../resources/rides.csv'
BOOTSTRAP_SERVERS = 'localhost:9092'

TOPIC_WINDOWED_VENDOR_ID_COUNT = 'vendor_counts_windowed'

PRODUCE_TOPIC_RIDES_CSV = CONSUME_TOPIC_RIDES_CSV = 'rides_csv'

RIDE_SCHEMA = T.StructType(
    [T.StructField("vendor_id", T.IntegerType()),
     T.StructField('tpep_pickup_datetime', T.TimestampType()),
     T.StructField('tpep_dropoff_datetime', T.TimestampType()),
     T.StructField("passenger_count", T.IntegerType()),
     T.StructField("trip_distance", T.FloatType()),
     T.StructField("payment_type", T.IntegerType()),
     T.StructField("total_amount", T.FloatType()),
     ])
```

Also in this pyspark folder, we will have a producer.py file with the following content (similar as before, just different encoder/decoder):

Imports first
```python
import csv
from time import sleep
from typing import Dict
from kafka import KafkaProducer

from settings import BOOTSTRAP_SERVERS, INPUT_DATA_PATH, PRODUCE_TOPIC_RIDES_CSV
```

RideCSVProducer class and initializer:
```python
class RideCSVProducer:
    def __init__(self, props: Dict):
        self.producer = KafkaProducer(**props)
        # self.producer = Producer(producer_props)
```

Read records method:
```python
@staticmethod
def read_records(resource_path: str):
	records, ride_keys = [], []
	i = 0
	with open(resource_path, 'r') as f:
		reader = csv.reader(f)
		header = next(reader)  # skip the header
		for row in reader:
			# vendor_id, passenger_count, trip_distance, payment_type, total_amount
			records.append(f'{row[0]}, {row[1]}, {row[2]}, {row[3]}, {row[4]}, {row[9]}, {row[16]}')
			ride_keys.append(str(row[0]))
			i += 1
			if i == 5:
				break
	return zip(ride_keys, records)
```

Publish method:
```python
def publish(self, topic: str, records: [str, str]):
	for key_value in records:
		key, value = key_value
		try:
			self.producer.send(topic=topic, key=key, value=value)
			print(f"Producing record for <key: {key}, value:{value}>")
		except KeyboardInterrupt:
			break
		except Exception as e:
			print(f"Exception while producing record - {value}: {e}")
	
	self.producer.flush()
	sleep(1)
```

and main function:

```python
if __name__ == "__main__":
    config = {
        'bootstrap_servers': [BOOTSTRAP_SERVERS],
        'key_serializer': lambda x: x.encode('utf-8'),
        'value_serializer': lambda x: x.encode('utf-8')
    }
    producer = RideCSVProducer(props=config)
    ride_records = producer.read_records(resource_path=INPUT_DATA_PATH)
    print(ride_records)
    producer.publish(topic=PRODUCE_TOPIC_RIDES_CSV, records=ride_records)
```

Make sure pyspark is pip installed.

Now we see when we run our producer, we get an output:
![[Screenshot 2024-03-11 at 1.33.42 PM.png]]

Now let's create the consumer.py file:

Imports first
```python
import argparse
from typing import Dict, List
from kafka import KafkaConsumer

from settings import BOOTSTRAP_SERVERS, CONSUME_TOPIC_RIDES_CSV
```

Create class and initializer:
```python
class RideCSVConsumer:
    def __init__(self, props: Dict):
        self.consumer = KafkaConsumer(**props)
```

Create consume_from_kafka method:

```python
def consume_from_kafka(self, topics: List[str]):
	self.consumer.subscribe(topics=topics)
	print('Consuming from Kafka started')
	print('Available topics to consume: ', self.consumer.subscription())
	while True:
		try:
			# SIGINT can't be handled when polling, limit timeout to 1 second.
			msg = self.consumer.poll(1.0)
			if msg is None or msg == {}:
				continue
			for msg_key, msg_values in msg.items():
				for msg_val in msg_values:
					print(f'Key:{msg_val.key}-type({type(msg_val.key)}), '
						  f'Value:{msg_val.value}-type({type(msg_val.value)})')
		except KeyboardInterrupt:
			break

	self.consumer.close()
```

And our main function:

```python
if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Kafka Consumer')
    parser.add_argument('--topic', type=str, default=CONSUME_TOPIC_RIDES_CSV)
    args = parser.parse_args()

    topic = args.topic
    config = {
        'bootstrap_servers': [BOOTSTRAP_SERVERS],
        'auto_offset_reset': 'earliest',
        'enable_auto_commit': True,
        'key_deserializer': lambda key: int(key.decode('utf-8')),
        'value_deserializer': lambda value: value.decode('utf-8'),
        'group_id': 'consumer.group.id.csv-example.1',
    }
    csv_consumer = RideCSVConsumer(props=config)
    csv_consumer.consume_from_kafka(topics=[topic])
```

Let's run our consumer and check out the output:
![[Screenshot 2024-03-11 at 1.37.48 PM.png]]

Okay now to connect pyspark with Kafka, let's do some prototyping in a notebook before we create our script. Go to localhost:8888 to open JupyterLab. There we can create a new notebook that has the following which sets an environment variable:

```python
import os
os.environ['PYSPARK_SUBMIT_ARGS'] = '--packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.3.1,org.apache.spark:spark-avro_2.12:3.3.1 pyspark-shell'
```

Now we create a SparkSession as we have seen in the previous module:

```python
from pyspark.sql import SparkSession
import pyspark.sql.types as T
import pyspark.sql.functions as F

spark = SparkSession \
    .builder \
    .appName("Spark-Notebook") \
    .getOrCreate()
```

Now we will read from Kafka:

```python
# default for startingOffsets is "latest"
df_kafka_raw = spark \
    .readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092,broker:29092") \
    .option("subscribe", "rides_csv") \
    .option("startingOffsets", "earliest") \
    .option("checkpointLocation", "checkpoint") \
    .load()
```

Let's check out the schema:
```python
df_kafka_raw.printSchema()
```
![[Screenshot 2024-03-11 at 1.53.32 PM.png]]
We can apply a SELECT statement:

```python
df_kafka_encoded = df_kafka_raw.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
```

and now the schema should be updated:
```python
df_kafka_encoded.printSchema()
```
![[Screenshot 2024-03-11 at 1.54.05 PM.png]]

This encoding is what we will use to create a structure streaming DataFrame with a specified schema:

```python
ride_schema = T.StructType(
    [T.StructField("vendor_id", T.IntegerType()),
     T.StructField("tpep_pickup_datetime", T.TimestampType()),
     T.StructField("tpep_dropoff_datetime", T.TimestampType()),
     T.StructField("passenger_count", T.IntegerType()),
     T.StructField("trip_distance", T.FloatType()),
     T.StructField("payment_type", T.IntegerType()),
     T.StructField("total_amount", T.FloatType()),
    ])
```

```python
def parse_ride_from_kafka_message(df_raw, schema):
	""" take a Spark Streaming df and parse value col based on <schema>, return streaming df cols in schema """
	assert df_raw.isStreaming is True, "DataFrame doesn't receive streaming data"

	df = df_raw.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

	# split attributes to nested array in one column
	col = F.split(df['value'], ', ')

	# expand col to multiple top-level columns
	for idx, field in enumerate(schema):
		df = df.withColumn(field.name, col.getItem(idx).cast(field.dataType))
	return df.select([field.name for field in schema])
```

Let's create a DataFrame with this schema:

```python
df_rides = parse_ride_from_kafka_message(df_raw, schema)
```

And check that it satisfies our schema:
```python
df_rides.printSchema()
```

![[Screenshot 2024-03-11 at 2.04.50 PM.png]]

Now in order to look at our DataFrame, we have to define our sink operation & streaming query through the writeStream command.

---
**Output Sink Options**
* File Sink: store the output to the directory
* Kafka Sink: stores the output to one or more topics in Kafka
* Foreach Stink:
* (for debugging) Console Sink, Memory Sink
---
There are three types of **Output Modes**:
* Complete: the whole result table will be outputted to the sink after every trigger. This is supported for aggregation queries.
* Append (default): only new rows are added to the result table
* Update: only updated rows are outputted
Output mode differs based on the set of transformations applied to the streaming data.
---
**Triggers**
The trigger settings of a streaming query define the timing of streaming data processing. Spark streaming support micro-batch streamings schema and you can select from the following options based on requirements:
* default-micro-batch-mode
* fixed-interval-micro-batch-mode
* one-time-micro-batch-mode
* available-now-micro-batch-mode
```python
def sink_console(df, output_mode: str = 'complete', processing_time: str = '5 seconds'):
	write_query = df.writeStream \
		.outputMode(output_mode) \
		.trigger(processingTime=processing_time) \
		.format("console") \
		.option("truncate", False) \
		.start()
	return write_query # pyspark.sql.streaming.StreamingQuery
```

Let's apply this to write our query:

```python
write_query = sink_console(df_rides, output_mode='append')
```

and now we can see our data:

![[Screenshot 2024-03-11 at 2.15.04 PM.png]]
Then if we run producer.py again we will get a second batch only showing us the new records:

![[Screenshot 2024-03-11 at 2.15.56 PM.png]]

We can stop writing the query:

```python
write_query.stop()
```

Now here we can try to save the output of the query to a dataframe.

```python
def sink_memory(df, query_name, query_template):
	write_query = df \
		.writeStream \
		.queryName(query_name) \
		.format('memory') \
		.start()
	query_str = query_template.format(table_name=query_name)
	query_results = spark.sql(query_str)
	return write_query, query_results
```
Let's see what this looks like:

```python
query_name = 'vendor_id_counts'
query_template = 'select count(distinct(vendor_id)) from {table_name}'
write_query, df_vendor_id_counts = sink_memory(df=df_rides, query_name=query_name, query_template=query_template)

df_vendor_id_counts.show()
```

![[Screenshot 2024-03-11 at 2.22.47 PM.png]]

Now we stop the query manually:
```python
write_query.stop()
```

And finally let's see what it looks like when our sink is a Kafka topic:

```python
def prepare_dataframe_to_kafka_sink(df, value_columns, key_column=None):
	columns = df.columns
	df = df.withColumn("value", F.concat_ws(', ',*value_columns))
	if key_column:
		df = df.withColumnRenamed(key_column, "key")
		df = df.withColumn("key", df.key.cast('string'))
	return df.select(['key', 'value'])

def sink_kafka(df, topic, output_mode='append'):
	write_query = df.writeStream \
		.format("kafka") \
		.option("kafka.bootstrap.servers", "localhost:9092,broker:29092") \
		.outputMode(output_mode) \
		.option("topic", topic) \
		.option("checkpointLocation", "checkpoint") \
		.start()
	return write_query
```

It is important to note that writeStream is always expecting a key and value column. We will see how this works in the script.

 Speaking of, let's create streaming.py with these imports:
```python
from pyspark.sql import SparkSession
import pyspark.sql.functions as F

from settings import RIDE_SCHEMA, CONSUME_TOPIC_RIDES_CSV, TOPIC_WINDOWED_VENDOR_ID_COUNT
```
We will have a read_from_kafka function as we have in our notebook:

```python
def read_from_kafka(consume_topic: str):
    # Spark Streaming DataFrame, connect to Kafka topic served at host in bootrap.servers option
    df_stream = spark \
        .readStream \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "localhost:9092,broker:29092") \
        .option("subscribe", consume_topic) \
        .option("startingOffsets", "earliest") \
        .option("checkpointLocation", "checkpoint") \
        .load()
    return df_stream
```

A parse_rides_from_kafka_message function as we saw in the notebook:

```python
def parse_ride_from_kafka_message(df, schema):
    """ take a Spark Streaming df and parse value col based on <schema>, return streaming df cols in schema """
    assert df.isStreaming is True, "DataFrame doesn't receive streaming data"

    df = df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

    # split attributes to nested array in one Column
    col = F.split(df['value'], ', ')

    # expand col to multiple top-level columns
    for idx, field in enumerate(schema):
        df = df.withColumn(field.name, col.getItem(idx).cast(field.dataType))
    return df.select([field.name for field in schema])
```

Our sink_console, sink_memory, and sink_kafka functions:

```python
def sink_console(df, output_mode: str = 'complete', processing_time: str = '5 seconds'):
	write_query = df.writeStream \
		.outputMode(output_mode) \
		.trigger(processingTime=processing_time) \
		.format("console") \
		.option("truncate", False) \
		.start()
	return write_query # pyspark.sql.streaming.StreamingQuery

def sink_memory(df, query_name, query_template):
	write_query = df \
		.writeStream \
		.queryName(query_name) \
		.format('memory') \
		.start()
	query_str = query_template.format(table_name=query_name)
	query_results = spark.sql(query_str)
	return write_query, query_results

def prepare_dataframe_to_kafka_sink(df, value_columns, key_column=None):
	columns = df.columns
	df = df.withColumn("value", F.concat_ws(', ',*value_columns))
	if key_column:
		df = df.withColumnRenamed(key_column, "key")
		df = df.withColumn("key", df.key.cast('string'))
	return df.select(['key', 'value'])

def sink_kafka(df, topic, output_mode='append'):
	write_query = df.writeStream \
		.format("kafka") \
		.option("kafka.bootstrap.servers", "localhost:9092,broker:29092") \
		.outputMode(output_mode) \
		.option("topic", topic) \
		.option("checkpointLocation", "checkpoint") \
		.start()
	return write_query
```

We will also have two functions which perform operations on our DataFrame:

```python
def op_groupby(df, column_names):
    df_aggregation = df.groupBy(column_names).count()
    return df_aggregation


def op_windowed_groupby(df, window_duration, slide_duration):
    df_windowed_aggregation = df.groupBy(
        F.window(timeColumn=df.tpep_pickup_datetime, windowDuration=window_duration, slideDuration=slide_duration),
        df.vendor_id
    ).count()
    return df_windowed_aggregation
```

Note that these returned objects must have a write_query associated with them to actually access the output.

Let's check out our main function:

```python
if __name__ == "__main__":
    spark = SparkSession.builder.appName('streaming-examples').getOrCreate()
    spark.sparkContext.setLogLevel('WARN')

    # read_streaming data
    df_consume_stream = read_from_kafka(consume_topic=CONSUME_TOPIC_RIDES_CSV)
    print(df_consume_stream.printSchema())

    # parse streaming data
    df_rides = parse_ride_from_kafka_message(df_consume_stream, RIDE_SCHEMA)
    print(df_rides.printSchema())

    sink_console(df_rides, output_mode='append')

    df_trip_count_by_vendor_id = op_groupby(df_rides, ['vendor_id'])
    df_trip_count_by_pickup_date_vendor_id = op_windowed_groupby(df_rides, window_duration="10 minutes",
                                                                 slide_duration='5 minutes')

    # write the output out to the console for debugging / testing
    sink_console(df_trip_count_by_vendor_id)
    # write the output to the kafka topic
    df_trip_count_messages = prepare_df_to_kafka_sink(df=df_trip_count_by_pickup_date_vendor_id,
                                                      value_columns=['count'], key_column='vendor_id')
    kafka_sink_query = sink_kafka(df=df_trip_count_messages, topic=TOPIC_WINDOWED_VENDOR_ID_COUNT)

    spark.streams.awaitAnyTermination()
```

Now to run this file we actually need to submit it as a spark job to the master.
Let's double check what our spark version is:

```bash
spark-submit --version
```

For me, it is 3.5.1. This means I pip install pyspark version 3.5.1 and then that version is used in the --packages flag in the spark-submit.sh file which will shorten the commands needed to run the streaming.py file with spark:

```bash
# Submit Python code to SparkMaster

if [ $# -lt 1 ]
then
	echo "Usage: $0 <pyspark-job.py> [ executor-memory ]"
	echo "(specify memory in string format such as \"512M\" or \"2G\")"
	exit 1
fi
PYTHON_JOB=$1

if [ -z $2 ]
then
	EXEC_MEM="1G"
else
	EXEC_MEM=$2
fi
spark-submit --master spark://localhost:7077 --num-executors 2 \
	           --executor-memory $EXEC_MEM --executor-cores 1 \
             --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.1,org.apache.spark:spark-avro_2.12:3.5.1,org.apache.spark:spark-streaming-kafka-0-10_2.12:3.5.1 \
             $PYTHON_JOB
```

So we can run the file by sending it to master:

```bash
chmod +x spark-submit.sh
./spark-submit.sh streaming.py
```
We see the schema printed:
![[Screenshot 2024-03-11 at 4.49.37 PM.png]]

The result of our query:
![[Screenshot 2024-03-11 at 4.49.58 PM.png]]
and our second query:
![[Screenshot 2024-03-11 at 4.50.18 PM.png]]
If we run producer.py again, we get the new rows:
![[Screenshot 2024-03-11 at 4.51.18 PM.png]]

If you delete the checkpoint folder, it will reset the tables back to nothing.

# Homework

## Redpanda version

Check the version of redpanda:
Get the docker container up and log into the shell:
```bash
docker-compose up
docker exec -it redpanda-1 bash
```

```bash
rpk version
>> v22.3.5 (rev 28b2443)
```

## Create a topic

```bash
rpk topic create test-topic
>>TOPIC       STATUS
test-topic  OK
```

## Connecting to Kafka

The output of producer.bootstrap_connected() is ```True```.

## Sending data in the stream
```python
t0 = time.time()
topic_name = 'test-topic'

for i in range(10):
	message = {'number': i}
	producer.send(topic_name, value=message)
	print(f"Sent: {message}")
	time.sleep(0.05)

t1 = time.time()
print(f'sending messages took {(t1 - t0):.2f} seconds')

t2 = time.time()
producer.flush()
t3 = time.time()

print(f'flushing took {(t3 - t2):.2f} seconds')
```
![[Screenshot 2024-03-12 at 4.20.44 PM.png]]

## Consuming data

In the container run:
```bash
rpk topic consume test-topic
```

![[Screenshot 2024-03-12 at 4.22.30 PM.png]]

## Sending the taxi data

Create a green-taxi topic in the container:

```bash
rpk topic create green-trips
```

Send the data from the green_tripdata csv file to the green-trips topic:

```python
import pandas as pd
df_green = pd.read_csv('green_tripdata_2019-10.csv.gz', compression='gzip')

df_green = df_green[['lpep_pickup_datetime',
	'lpep_dropoff_datetime',
	'PULocationID',
	'DOLocationID',
	'passenger_count',
	'trip_distance',
	'tip_amount']]

topic_name = 'green-trips'
t0 = time.time()
for row in df_green.itertuples(index=False):
	row_dict = {col: getattr(row, col) for col in row._fields}
	producer.send(topic_name, value=row_dict)
t1 = time.time()

print(f'Sending these messages took {(t1 - t0):.2f} seconds')
```
![[Screenshot 2024-03-12 at 6.00.13 PM.png]]

## PySpark Consumer

Here is the new row after the schema:

```bash
Row(lpep_pickup_datetime='2019-10-01 00:26:02', lpep_dropoff_datetime='2019-10-01 00:39:58', PULocationID=112, DOLocationID=196, passenger_count=1.0, trip_distance=5.88, tip_amount=0.0)
```
## Most popular destination

Update our producer to include timestamp:

```python
for row in df_green.itertuples(index=False):
	row_dict = {col: getattr(row, col) for col in row._fields}
	row_dict["timestamp"] = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
	producer.send(topic_name, value=row_dict)
```

Update stream read in consumer to be latest:

```python
.option("startingOffsets", "latest")
```

Update schema in consumer with the following:

```python
.add("timestamp", types.TimestampType())
```

Create query and write it:

```python
popular_destinations = (
	green_stream.groupBy(
		F.window(timeColumn=green_stream.timestamp, windowDuration="5 minutes"),
		green_stream.DOLocationID,
	)
	.count()
	.orderBy(F.desc("count"))
)

query = (
	popular_destinations.writeStream.outputMode("complete")
	.format("console")
	.option("truncate", "false")
	.start()
)

query.awaitTermination()
```

Run the producer and here is the output:

![[Screenshot 2024-03-12 at 8.08.29 PM.png]]

Most population location is that with ID 74.