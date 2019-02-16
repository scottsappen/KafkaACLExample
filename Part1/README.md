# KafkaACLExample

**Part 1<br/>
Adjust brokers to support ACLs**

Modify your broker server properties file to work with ACLs.

The super.users allows any user defined in that list to have full admin access to the cluster.

By setting allow.everyone.if.no.acl.found to false, we are denying access to any user that is not part of an ACL. Note that other components may fail until you configure them appropriately (e.g. schema registry).

We are going to work with the keytabs created in the MITKerberosExample repo, but you can use whatever you want.

```
sudo vi ../etc/kafka/server.properties

authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
super.users=User:admin;User:ubuntu
allow.everyone.if.no.acl.found=false
```

Now restart your brokers.

```
./confluent start zookeeper

KAFKA_OPTS="-Djava.security.auth.login.config=/home/ubuntu/confluent-5.1.1/etc/kafka/kafka_server_jaas.conf" ./confluent start kafka
```

Now you can move to Part 2 to test our your new changes.
