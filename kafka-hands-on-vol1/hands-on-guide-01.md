첫번째 실습.


step00. download file kafka-hands-on-vol1/step.00/kafka_2.13-3.2.1.tgz
```shell
$ tar -xzf kafka_2.13-3.2.1.tgz
$ cd kafka_2.13-3.2.1
```

step01. START THE KAFKA ENVIRONMENT  
터미널로 Zookeeper 서비스 실행  
https://zookeeper.apache.org/  
ZooKeeper는 구성 정보 유지 관리, 이름 지정, 분산 동기화 제공 및 그룹 서비스 제공을 위한 중앙 집중식 서비스
```shell
# Start the ZooKeeper service
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

다른 세션 터미널 탭으로 kafka 서버 실행
```shell
# Start the Kafka broker service
$ bin/kafka-server-start.sh config/server.properties
```

step02.   CREATE A TOPIC TO STORE YOUR EVENTS  
다른 터미널 세션을 열고 다음을 실행
```shell
$ bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```

arguments 없이 kafka-topics.sh 명령을 실행하여 사용 정보를 표시 가능.  
예를 들어 새 주제의 파티션 수와 같은 세부 정보도 표시
```shell
$ bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
Topic:quickstart-events  PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: quickstart-events Partition: 0    Leader: 0   Replicas: 0 Isr: 0
```

step03. WRITE SOME EVENTS INTO THE TOPIC  
다른 터미널 세션을 열고 다음을 실행  
Kafka 클라이언트는 이벤트 쓰기(또는 읽기)를 위한 네트워크를 통해 Kafka 브로커와 통신합니다.  
일단 수신되면 브로커는 필요한 기간 동안(심지어 영원히) 내구성 및 fault-tolerant 방식으로 이벤트를 저장합니다.
```shell
$ bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
#type anything
> dxdev3
> latework is not good for your health
> best worker automation
```
You can stop the producer client with Ctrl-C at any time.  

step04. READ THE EVENTS
다른 터미널 세션을 열고 다음을 실행  
```shell
$ bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
> dxdev3
> latework is not good for your health
> best worker automation
```

step05.  IMPORT/EXPORT YOUR DATA AS STREAMS OF EVENTS WITH KAFKA CONNECT    
config/connect-standalone.properties 파일을 편집하고 다음과 일치하는 plugin.path 구성 속성을 추가 또는 변경하고 파일을 저장합니다.
```shell
> echo "plugin.path=libs/connect-file-3.2.1.jar"
```
그런 다음 테스트할 시드 데이터를 생성하여 시작합니다.
```shell
> echo -e "foo\nbar" > test.txt
```
다음으로 standalone 모드에서 실행되는 두 개의 커넥터를 시작합니다. 즉, 단일 로컬 전용 프로세스에서 실행됩니다.
```shell
> bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```

test.sink.txt 파일을 확인!!
```shell 
> more test.sink.txt
foo
bar
```
콘솔 소비자를 실행하여 topic 데이터  확인 가능
```shell
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
...
```

새탭을 열어 아래 명령어를 실행하면 파일에 저장된 부분을 동적으로 읽음을 알 수 있다.
```
> echo Another line>> test.txt
```

step06. TERMINATE THE KAFKA ENVIRONMENT  
모든 프로세스를 ctrl+c 로 종료할 것.
그 후 도중에 생성한 이벤트를 포함하여 로컬 Kafka 환경의 데이터도 삭제하려면 다음 명령을 실행합니다.
```shell
$ rm -rf /tmp/kafka-logs /tmp/zookeeper
```
