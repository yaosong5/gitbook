报错

INFO Replica loaded for partition __consumer_offsets-13 with initial high watermark 0 (kafka.cluster.Replica)
[2018-08-13 18:41:51,789] ERROR Error while creating log for __consumer_offsets-13 in dir /usr/kafka/kafkalogs (kafka.server.LogDirFailureChannel)
java.io.IOException: Invalid argument

at sun.nio.ch.FileChannelImpl.map0(Native Method)

打开zookeeper 查看

```
ls /

删除这些节点
rmr /cluster
rmr /brokers
rmr /isr_change_notification
rmr /log_dir_event_notification
rmr /controller_epoch
rmr /kafka-manager
rmr /consumers
rmr /latest_producer_id_block
rmr /config




rm -rf /Users/yaosong/Yao/DockerSource/sparkSource/kafkaall/kafka3/kafkalogs
rm -rf /Users/yaosong/Yao/DockerSource/sparkSource/kafkaall/kafka3/logs
rm -rf /Users/yaosong/Yao/DockerSource/sparkSource/kafkaall/kafka1/kafkalogs
rm -rf /Users/yaosong/Yao/DockerSource/sparkSource/kafkaall/kafka1/logs
rm -rf /Users/yaosong/Yao/DockerSource/sparkSource/kafkaall/kafka2/kafkalogs
rm -rf /Users/yaosong/Yao/DockerSource/sparkSource/kafkaall/kafka2/logs
```

