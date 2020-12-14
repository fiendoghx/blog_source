
官方文档：https://kafka.apache.org/0110/documentation.html#security_client_staticjaas


第一步：建立zookeeper鉴权配置 /opt/kafka/zookeeper.jaas.conf
Server {
	org.apache.kafka.common.security.plain.PlainLoginModule required
	username="admin"
	password="admin"
	user_admin="admin";
};
第二步：建立broker节点鉴权配置 /opt/kafka/kafka.jaas.conf
Client {
	org.apache.kafka.common.security.plain.PlainLoginModule required
	username="admin"
	password="admin";
};
 
KafkaServer {
	org.apache.kafka.common.security.plain.PlainLoginModule required
	username="admin"
	password="admin"
	user_admin="admin"
	user_alice="alice";
};
PS： user_alice="alice";  表示 username为alice，password为alice的一个账户
第三步：建立客户端鉴权配置 /opt/kafka/client.jaas.conf

Client {
	org.apache.kafka.common.security.plain.PlainLoginModule required
	username="admin"
	password="admin";
};
第四步：修改zookeeper.properties，添加如下内容：
listeners=SASL_PLAINTEXT://host.name:port
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.enabled.mechanisms=PLAIN  
sasl.mechanism.inter.broker.protocol=PLAIN 

第五步：修改server.properties，添加如下内容：
authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider
requireClientAuthScheme=sasl
jaasLoginRenew=3600000

第六步：修改zookeeper-server-start.sh 加入
if [ "x$KAFKA_OPTS" ]; then
   export KAFKA_OPTS=-Djava.security.auth.login.config=/opt/kafka/zookeeper.jaas.conf
fi

第七步: 修改kafka-server-start.sh 加入
if [ "x$KAFKA_OPTS" ]; then
   export KAFKA_OPTS=-Djava.security.auth.login.config=/opt/kafka/kafka.jaas.conf
fi

第八步: 修改kafka-console-consumer.sh/kafka-console-producer.sh 加入
if [ "x$KAFKA_OPTS" ]; then
   export KAFKA_OPTS=-Djava.security.auth.login.config=/opt/kafka/client.jaas.conf
fi

第九步：启动zookeeper、启动kafka
./zookeeper-server-start.sh -daemon ../conf/zookeeper.properties

./kafka-server-start.sh -daemon ../conf/server.properties

第十步：启动producer和consumer验证
./kafka-console-producer --broker-list host.name:port --topic test_sasl --producer.config /opt/kafka/producer.properties

./kafka-console-consumer --bootstrap-server host.name:port --topic test_sasl  --consumer.config /opt/kafka/consumer.properties --from-beginning


PS:
java中连接kafka配置：
System.setProperty("java.security.auth.login.config","/opt/kafka/kafka_client_jass.conf");
props.put("security.protocol","SASL_PLAINTEXT");
props.put("sasl.mechanism","PLAIN");
props.put("sasl.jaas.config",
		"org.apache.kafka.common.security.plain.PlainLoginModule required username=\"admin\" password=\"admin\";");
