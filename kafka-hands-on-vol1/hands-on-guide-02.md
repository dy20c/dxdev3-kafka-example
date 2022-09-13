두번째 실습.
RUN KAFKA STREAMS DEMO APPLICATION

라이브러리안에 WordCountDemo를 활용한 실습 (demo code 아래 참조)
``` 
// Serializers/deserializers (serde) for String and Long types
final Serde<String> stringSerde = Serdes.String();
final Serde<Long> longSerde = Serdes.Long();

// Construct a `KStream` from the input topic "streams-plaintext-input", where message values
// represent lines of text (for the sake of this example, we ignore whatever may be stored
// in the message keys).
KStream<String, String> textLines = builder.stream(
      "streams-plaintext-input",
      Consumed.with(stringSerde, stringSerde)
    );

KTable<String, Long> wordCounts = textLines
    // Split each text line, by whitespace, into words.
    .flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))

    // Group the text words as message keys
    .groupBy((key, value) -> value)

    // Count the occurrences of each word (message key).
    .count();

// Store the running counts as a changelog stream to the output topic.
wordCounts.toStream().to("streams-wordcount-output", Produced.with(Serdes.String(), Serdes.Long()));

```

step00.Start the Kafka server  
```shell
> bin/zookeeper-server-start.sh config/zookeeper.properties
```

```shell
bin/kafka-server-start.sh config/server.properties
```

step01. Prepare input topic and start Kafka producer  
다음으로 streams-plaintext-input이라는 input topic과 streams-wordcount-output이라는 output topic을 만듭니다.
```shell
> bin/kafka-topics.sh --create \
    --bootstrap-server localhost:9092 \
    --replication-factor 1 \
    --partitions 1 \
    --topic streams-plaintext-input
Created topic "streams-plaintext-input".
```
```shell
> bin/kafka-topics.sh --create \
    --bootstrap-server localhost:9092 \
    --replication-factor 1 \
    --partitions 1 \
    --topic streams-wordcount-output \
    --config cleanup.policy=compact
Created topic "streams-wordcount-output".
```

kafka topic check
```shell
> bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe
```

step02. Start the Wordcount Application
```shell
> bin/kafka-run-class.sh org.apache.kafka.streams.examples.wordcount.WordCountDemo
```
데모 애플리케이션은 입력 주제 streams-plaintext-input에서 읽고,  
각 읽기 메시지에 대해 WordCount 알고리즘 계산을 수행하고,  
현재 결과를 출력 주제 streams-wordcount-output에 계속 씁니다.   
따라서 결과가 Kafka에 다시 기록되므로 로그 항목을 제외하고 STDOUT 출력이 없습니다.  

이제 별도의 터미널에서  console producer를 시작하여 이 topic에 대해 일부 입력 데이터를 쓸 수 있습니다.
```shell
> bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic streams-plaintext-input
```
그리고 별도의 터미널에서 console consumer로 부터 output topic을 읽어 WordCount 데모 응용 프로그램의 출력을 검사합니다.
```shell
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
    --topic streams-wordcount-output \
    --from-beginning \
    --formatter kafka.tools.DefaultMessageFormatter \
    --property print.key=true \
    --property print.value=true \
    --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
    --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer
```

step03.  Process some data
이제 한 줄의 텍스트를 입력하고 <RETURN> 키를 눌러 입력 주제 streams-plaintext-input에 console producer와 함께 메시지를 작성해 보겠습니다. 이렇게 하면 메시지 키가 null이고 메시지 값이 방금 입력한 문자열 인코딩된 텍스트 줄인 입력 topic에 새 메시지가 전송됩니다.
```shell
> bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic streams-plaintext-input
dxdev3 who join us 
```

이 메시지는 Wordcount 애플리케이션에 의해 처리되고 다음 output 데이터는 streams-wordcount-output topic에 writed되고 consoule consumer가 print합니다.
```shell
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
    --topic streams-wordcount-output \
    --from-beginning \
    --formatter kafka.tools.DefaultMessageFormatter \
    --property print.key=true \
    --property print.value=true \
    --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
    --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer

dxdev3	1
who	1
join	1
us	    1
```

여기서는 저의 말에 따라 더 실습할거에요.

step04. 
ctrl+c 로 종료