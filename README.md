# light-email
An email sender based on Kafka Streams for Event Sourcing

Most applications have email sending features, and it is a common practice to send mail from the application code directly with a utility. If there are multiple applications sending emails at the same time synchronously, it is very easy to overload the email server. At the same time, it will slow down the applications while they are waiting for the email server to accept the requests. 

In Event Sourcing applications, email sending is a side effect if events trigger it. We want to have a simple way to bypass email sending if the system is trying to replay the existing events. 

The taiji-blockchain has a service called KYC (Know Your Client), and it needs to send out emails to verify the users' email addresses. Given the above issues with the existing solutions, we are planning to build a new email sender based on Kafka Streams. 

### Features

* Send email as part of the streams processing so that we can control the number of email requests to the mail server from the number of microservices instances that are responsible for sending emails. 

* Every email event has a unique id and this id will ensure that the same email will only be sent once. When the event sourcing application is replayed, no emails will be sent out, so there are no side effects. 

* Cache the email server connection to increase the throughput of the emails without putting a lot of pressure on the email server.

* Multiple senders email addresses are supported. 

* Automatically load balanced by the Kafka partitions.

* High availability supported by the Kafka partitions. 

* The Kafka topic is configurable so that it can be named anything to follow the convention of each organization. 

* Support aggregate of events and send one email in a batch for example per 30 minutes window.

### Components

* email-event

As it is an event sourcing application, all events are defined in this module with avro schema and generated events.

* email-sender

A Kafka streams micorservice that is responsible for process the email event stream and send out emails. 

* email-producer

A Kafka producer microservice that is responsible for accepting rest email request and put into the email topic on Kafka. 

### Topics

By default, we have two topics that for emails. 

* eamil-realtime is a topic that you send event to and the email sender will pick up messages and send out emails immediately. 

* email-batch is a topic that you can push multiple events to the topic and an aggregated email will be sent out every 30 minutes.

