# KafkaACLExample

This is a sample repo showing how to use ACLs in Kafka.

You need to have your clients that are authenticated to the cluster have the right set of permissions (these are ACLs or access control lists).

In this example, we will use simple consumer and producers using keytabs generated from the MITKerberosExample repo.

If you need help configuring authentication, see the following repos:
- If you want to setup SASL_SSL - https://github.com/scottsappen/KafkaSASL_SSLAuthExample
- If you want to setup SSL - https://github.com/scottsappen/KafkaSSLAuthExample
- If you want to create your own keytabs - https://github.com/scottsappen/MITKerberosExample

Think of ACLs in the following ways:
- Topic ACLs to restrict or allow which clients can read/write data
- Consumer Group ACLs to define which clients can use a specific consumer Group
- Cluster ACLs to define which clients can perform operations like topic creation

Note a few things:
- There is a special "Super User" in Kafka that can do everything without any ACL
- There is no concept of a "User Group" in Kafka. That is to say, ACLs are defined for each client, not at a "user group" level
- ACLs are stored in ZooKeeper so it's critical you restrict who has access to ZK through security or network rules (hence only Kafka admins have the ability to work with ACLs in ZK)

**What we will do**

See the subdirectories in this repo for the details of each part:

- Part 1 - Adjust brokers to support ACLs
- Part 2 - Use clients to run some tests

That's it. When you're done, I hope you will have enjoyed this walkthrough and learned something new.
