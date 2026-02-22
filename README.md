
| WebSite   | REPO                           |
|-----------|--------------------------------|
| GitHub    | https://github.com/nishakar123 |
| BitBucket | https://bitbucket.org/nishakar123 |


## Service Port
* api-gateway : 9091
* auth2-service : 9000
* auth-service : 9001
* config-server : 8888
* file-handling-service : 8081
* registry-service : 8761
* spring-boot-new-features : 8080
* vehicle-parking-service : 8082
* kafka-producer : 8083
* kafka-consumer : 8084
---
## To see tree structure of project
* Install tree first on Windows
> winget install tree
* run below cmd from root project to see project tree structure
> tree /F
---
## Kafka Installation Guide
### ⚠️ IMPORTANT (Kafka 4.x KRaft not Zookeeper)
## Error during Kafka Setup
### Error 1:
🚨 Kafka on Windows Rule
Kafka MUST NOT be installed in a directory that contains spaces
* Wrong: D:\Softwares\Program Files\kafka\bin\windows
* Correct: D:\Softwares\kafka\bin\windows
---
### Error 2:
```sh
The input line is too long.
The syntax of the command is incorrect.
```
✅ Fix #1 (Correct Way) — Use PowerShell Instead of CMD
Kafka 4.x on Windows must be run from PowerShell
▶️ Step 1 — Open PowerShell as Administrator
Right Click → Run as Administrator
then go to directory to run the required cmd
---
### Error 3:
```sh
main ERROR Reconfiguration failed: No configuration found for '764c12b6'
```
```sh
2026-02-19T12:23:53.750035900Z main ERROR Reconfiguration failed: No configuration found for '764c12b6' at 'null' in 'null'
Bootstrap metadata: BootstrapMetadata(records=[ApiMessageAndVersion(FeatureLevelRecord(name='metadata.version', featureLevel=29) at version 0), ApiMessageAndVersion(FeatureLevelRecord(name='eligible.leader.replicas.version', featureLevel=1) at version 0), ApiMessageAndVersion(FeatureLevelRecord(name='group.version', featureLevel=1) at version 0), ApiMessageAndVersion(FeatureLevelRecord(name='share.version', featureLevel=1) at version 0), ApiMessageAndVersion(FeatureLevelRecord(name='streams.version', featureLevel=1) at version 0), ApiMessageAndVersion(FeatureLevelRecord(name='transaction.version', featureLevel=2) at version 0)], metadataVersionLevel=29, source=format command)
Formatting metadata directory D:/Softwares/kafka/kraft-combined-logs with metadata.version 4.2-IV1
```
* ✅ This is actually SUCCESS — not an error
  People get confused because of this log line 👇
  main ERROR Reconfiguration failed: No configuration found for '764c12b6'
  This is coming from Log4j dynamic reconfiguration
  👉 NOT from Kafka storage formatting

✅ Important Line in Your Output
>Formatting metadata directory D:/Softwares/kafka/kraft-combined-logs
with metadata.version 4.2-IV1

💥 This means:
* Cluster UUID applied
* Metadata log created
* KRaft storage formatted successfully
* Controller quorum config accepted
* You are READY to start Kafka
---
### Error 4:
>Because controller.quorum.voters is not set on this controller, you must specify one of the following: --standalone, --initial-controllers, or --no-initial-controllers.

>Required for KRaft mode
```sh
process.roles=broker,controller
node.id=1
controller.listener.names=CONTROLLER
listeners=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093
listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
controller.quorum.voters=1@localhost:9093
inter.broker.listener.name=PLAINTEXT
```
---
### Error 5:
> 2026-02-19T12:58:32.752888300Z main ERROR Reconfiguration failed: No configuration found for '764c12b6' at 'null' in 'null'

👉 This is NOT a Kafka error
This is coming from:
Log4j2 Dynamic Logger Reconfiguration

Kafka 4.x internally tries to reload logging config dynamically and prints this when no runtime config exists.

It does NOT affect:
* Broker startup
* Topic creation
* Producer send
* Consumer poll
* Controller quorum
* Metadata log
* Network communication
* So you can safely ignore it ✅
---
## Steps to install Kafka in Windows
Download the Kafka from here: [https://kafka.apache.org/community/downloads/]
##### Step1: Extract the downloaded file in directory(D:\Softwares\kafka).
##### Step2: Change the **log.dirs** directory in below properties file
    D:\kafka\config\
    * broker.properties (log.dirs=D:/Softwares/kafka/kraft-broker-logs)
    * controller.properties(log.dirs=D:/Softwares/kafka/kraft-controller-logs)
    * server.properties(log.dirs=D:/Softwares/kafka/kraft-combined-logs)
##### Step3: Generate a Cluster UUID
>      D:\kafka\bin\windows> .\kafka-storage.bat random-uuid
RTe9vR2iT8-OgmilRLAy_A - Generated UUID
##### Step4: Format Log Directories
```sh
      D:\Softwares\kafka\bin\windows> .\kafka-storage.bat format -t RTe9vR2iT8-OgmilRLAy_A -c ..\..\config\server.properties
```
##### Step5: Start the Kafka Server
>       D:\Softwares\kafka\bin\windows> .\kafka-server-start.bat ..\..\config\server.properties
##### Step6: Open new terminal to Create a topic to store your events
>      D:\Softwares\kafka\bin\windows>kafka-topics.bat --create --topic quickstart-events --bootstrap-server localhost:9092
>      Created topic quickstart-events.
##### Step7: Describe created topic to see details
>      D:\Softwares\kafka\bin\windows>kafka-topics.bat --describe --topic quickstart-events --bootstrap-server localhost:9092

>Topic: quickstart-events        TopicId: 1zN1XrUEQiyFUuXgD7KEFw PartitionCount: 1       ReplicationFactor: 1  Configs: min.insync.replicas=1,segment.bytes=1073741824
Topic: quickstart-events        Partition: 0    Leader: 1       Replicas: 1     Isr: 1  Elr: LastKnownElr:
##### Step8: Write some events into the topic
>       D:\Softwares\kafka\bin\windows>.\kafka-console-producer.bat --topic quickstart-events --bootstrap-server localhost:9092
>hi
>hello
##### Step9: Open new terminal to Read the events
>       D:\Softwares\kafka\bin\windows>.\kafka-console-consumer.bat --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
>hi
>hello
##### Step10: List all topics
>       D:\Softwares\kafka\bin\windows>.\kafka-topics.bat --bootstrap-server localhost:9092 --list
     __consumer_offsets
     quickstart-events
     test-topic
##### Step11: Terminate the Kafka environment
* Now that you reached the end of the quickstart, feel free to tear down the Kafka environment—or continue playing around.
    * Stop the producer and consumer clients with Ctrl-C, if you haven't done so already.
    * Stop the Kafka broker with Ctrl-C.

