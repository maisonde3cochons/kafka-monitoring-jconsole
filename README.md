## [JConsole을 이용한 Apache Kafka Monitoring 방법]

<br><br>

### [환경 구성도]
![image](https://user-images.githubusercontent.com/30817824/172519504-12f598b8-ab80-4c0f-8497-cb96ff9b393c.png)



<br><br>


> ##  STG.01 GCP VM Instance 생성

<br>

###  GCP에 접속하여 VM Instance를 생성한다

![image](https://user-images.githubusercontent.com/30817824/172508942-a7789e82-080d-48b3-b6d6-914b00ae8b48.png)

<br>

### broker-01이라는 이름의 인스턴스를 생성(Seoul Region > e2-standard-4)

<br>

![image](https://user-images.githubusercontent.com/30817824/172509365-2958fd8a-58ee-49ff-a25d-cba5ff3abf81.png)

<br>

### Boot Disk는 Centos로 선택

<br>

![image](https://user-images.githubusercontent.com/30817824/172509380-4ad29e2c-00ba-4ed0-aee9-f5ab8a5f5f1b.png)

<br><br>

> ## STG.02. Install & Configure

<br>

### Java 설치 및 JAVA_HOME 설정
```
sudo yum install -y java

# 현재 OS 설정이 한글인지 영어인지 확인한다. 
alternatives --display java

# 아래와 같이 출력되면 한글임. 
슬레이브 unpack200.1.gz: /usr/share/man/man1/unpack200-java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64.1.gz
현재 '최고' 버전은 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre/bin/java입니다.

### 한글인 경우 
alternatives --display java | grep '현재 /'| sed "s/현재 //" | sed 's|/bin/java로 링크되어 있습니다||'
export JAVA_HOME=$(alternatives --display java | grep '현재 /'| sed "s/현재 //" | sed 's|/bin/java로 링크되어 있습니다||')

### 영문인 경우
alternatives --display java | grep current | sed 's/link currently points to //' | sed 's|/bin/java||' | sed 's/^ //g'
export JAVA_HOME=$(alternatives --display java | grep current | sed 's/link currently points to //' | sed 's|/bin/java||' | sed 's/^ //g')

# 제대로 java 경로가 설정되었는지 확인
echo $JAVA_HOME
echo "export JAVA_HOME=$JAVA_HOME" >> ~/.bash_profile
source ~/.bash_profile
```

### Download apache kafka
```
cd ~
curl -O https://shared-penguin.s3.ap-northeast-2.amazonaws.com/Shared/kafka_zk.tar.gz
tar xvf kafka_zk.tar.gz
rm -rf kafka_zk.tar.gz
```

#### Run zookeeper
```
./start_zk.sh
```

#### Run kafka broker
```


## producer 또는 consumer에서 broker에 접속할 때 사용 가능한 hostname(GCP VM Instance 생성 시 지정한 Name) 설정 

> vi kafka/config/server.properties
advertised.listeners=PLAINTEXT://broker-01:9092

## java.rmi.server.hostname은 본인의 External IP(GCP에서 확인)로 변경
### rmi.server.hostname을 설정해야, 외부의 jconsole과 같은 도구에서 접속 가능
> export KAFKA_JMX_OPTS='-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false 
  -Dcom.sun.management.jmxremote.ssl=false 
  -Dcom.sun.management.jmxremote.port=9999 
  -Dcom.sun.management.jmxremote.rmi.port=9999 
  -Djava.rmi.server.hostname=34.64.37.29'

## kafka 기동
cd ~
./start_kafka.sh
```

#### Creeate a topic(kafka-mon)
```
> cd ~/kafka
> bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic redpolex-topic \
--partitions 2 --replication-factor 1

# 생성된 topic 확인
> bin/kafka-topics.sh --list --bootstrap-server localhost:9092 
redpolex-topic
```

#### GCP VPC Network Firewall 설정 (9999 port 허용)

![image](https://user-images.githubusercontent.com/30817824/172518925-0a960d8a-8a56-4d92-9af6-bf24231fcf53.png)

![image](https://user-images.githubusercontent.com/30817824/172518946-6441c4be-f390-4cba-a2b7-b78768e90f4e.png)



#### JConsole에서 확인

- VS Code Terminal에서 jdk/bin dir에 있는 jconsole.exe 실행 (jdk 설치 후)

![image](https://user-images.githubusercontent.com/30817824/172519146-4645b5d8-b14c-4c76-a70f-5f1acd2ca8fe.png)


```
./jconsole.exe
```

#### JConsole 접속 (GCP VM Instance의 External IP + :9999 포트 입력)

![image](https://user-images.githubusercontent.com/30817824/172519694-79d7dbd1-4139-47c6-aedc-c34741b723f3.png)


#### MBeans에서 Kafka의 각종 metrics들을 확인할 수 있다

![image](https://user-images.githubusercontent.com/30817824/172520130-99e06b6f-02e7-4d1f-beca-1af3d0f18f01.png)

