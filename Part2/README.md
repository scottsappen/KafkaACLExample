# KafkaACLExample

**Part 1<br/>
Use clients to run some tests**

Keep in mind if you use the same terminal window in various parts of this, you may need to clear out your KAFKA_OPTS.
```
echo KAFKA_OPTS=
```

Let's create a new topic to test with.

```
./kafka-topics --zookeeper <your zookeeper server>:2181 --create --topic acl-test --replication-factor 1 --partitions 1
```

Let's set a ACL for the read permission on the topic for a couple of the principals. We don't have any named consumer groups right now so we allow all consumer groups.

```
./kafka-acls --authorizer-properties zookeeper.connect=<your zookeeper server>:2181 --add --allow-principal User:reader -allow-principal User:writer --operation Read --group=* --topic acl-test
```

You should see someting that resembles:

```
Adding ACLs for resource `Topic:LITERAL:acl-test`:
 	User:reader has Allow permission for operations: Read from hosts: *
	User:writer has Allow permission for operations: Read from hosts: *

Adding ACLs for resource `Group:LITERAL:*`:
 	User:reader has Allow permission for operations: Read from hosts: *
	User:writer has Allow permission for operations: Read from hosts: *

Current ACLs for resource `Topic:LITERAL:acl-test`:
 	User:reader has Allow permission for operations: Read from hosts: *
	User:writer has Allow permission for operations: Read from hosts: *

Current ACLs for resource `Group:LITERAL:*`:
 	User:reader has Allow permission for operations: Read from hosts: *
	User:writer has Allow permission for operations: Read from hosts: *
```

Let's follow suit for additional permissions.

```
./kafka-acls --authorizer-properties zookeeper.connect=<your zookeeper server>:2181 --add --allow-principal User:writer --operation Write --topic acl-test
```

Let's see what the current permissions are on the topic.

```
./kafka-acls --authorizer-properties zookeeper.connect=<your zookeeper server>:2181 --list --topic acl-test
```

You should see something that resembles:

```
Current ACLs for resource `Topic:LITERAL:acl-test`:
 	User:reader has Allow permission for operations: Read from hosts: *
	User:writer has Allow permission for operations: Read from hosts: *
	User:writer has Allow permission for operations: Write from hosts: *
```

Let's start a producer as writer.

```
export KAFKA_OPTS="-Djava.security.auth.login.config=/tmp/kafka_client_jaas.conf"

kdestroy

kinit -kt /tmp/writer.user.keytab writer

./kafka-console-producer --broker-list <your kafka server>:9094 --topic acl-test --producer.config /tmp/kafka_client_kerberos.properties
```

Now let's start a consumer as reader.

```
export KAFKA_OPTS="-Djava.security.auth.login.config=/tmp/kafka_client_jaas.conf"

kdestroy

kinit -kt /tmp/reader.user.keytab reader

./kafka-console-consumer --bootstrap-server <your kafka server>:9094 --topic acl-test --consumer.config /tmp/kafka_client_kerberos.properties
```

Success!

Now let's do another test. Let's remove reader's permissions to see what happens (do this while the consumer is still running).

```
./kafka-acls --authorizer-properties zookeeper.connect=<your kafka server>:2181 --remove --allow-principal User:reader --operation Read --topic acl-test
```

Your consumer will exit with output that resembles:

```
[2019-02-16 20:27:23,097] WARN [Consumer clientId=consumer-1, groupId=console-consumer-22078] Not authorized to read from topic acl-test. (org.apache.kafka.clients.consumer.internals.Fetcher)
[2019-02-16 20:27:23,098] ERROR Error processing message, terminating consumer process:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.TopicAuthorizationException: Not authorized to access topics: [acl-test]
```

Now let's see what the behavior is for one of our super users. Start a producer with an admin keytab.

```
kinit -kt /tmp/admin.user.keytab admin

export KAFKA_OPTS="-Djava.security.auth.login.config=/tmp/kafka_client_jaas.conf"

./kafka-console-producer --broker-list <your kafka server>:9094 --topic acl-test --producer.config /tmp/kafka_client_kerberos.properties
```

And now start a consumer with an admin keytab.

```
kinit -kt /tmp/admin.user.keytab admin

export KAFKA_OPTS="-Djava.security.auth.login.config=/tmp/kafka_client_jaas.conf"

./kafka-console-consumer --bootstrap-server <your kafka server>:9094 --topic acl-test --consumer.config /tmp/kafka_client_kerberos.properties
```

Success! If you have trouble debugging, check out the kafka-authorizer.log for failures (denied access). You may need to adjust your log4j.properties logging level (DEBUG instead of INFO) to view authorizer success logging entries.
