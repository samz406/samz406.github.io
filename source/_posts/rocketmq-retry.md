---
title: RocketMQ消息重试
date: 2022-05-26 15:28:16
tags:
---



先看一个列子

**Producer**

```java
public class OrderMessage {


    public static void main(String[] args) throws MQBrokerException, RemotingException, InterruptedException, MQClientException {

        TransactionMQProducer producer = new TransactionMQProducer("testOrderMsgGroup");

        producer.setNamesrvAddr("192.192.192.61:9876");
        producer.start();
        MsgSelector msgSelector = new MsgSelector();

        for (int i = 0; i < 1; i++) {
            int orderId = i % 10;
            String msg = "Hello RocketMQ" + i;
            Message message = new Message("testOrderMsg", msg.getBytes());
            final SendResult send = producer.send(message, msgSelector, orderId);

            System.out.println(send);
        }

    }

    /**
     * 发送队列选择器
     */
    static class MsgSelector implements MessageQueueSelector {

        @Override
        public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {

            Integer id = (Integer) arg;
            int index = id % mqs.size();
            return mqs.get(index);
        }
    }
}

```



**Consumer**

```
public class OrderMsgConsumer {

    public static void main(String[] args) throws MQClientException {

        DefaultMQPushConsumer defaultMQPushConsumer = new DefaultMQPushConsumer("ProducerGroup");

        defaultMQPushConsumer.setNamesrvAddr("192.192.192.61:9876");
        defaultMQPushConsumer.subscribe("testOrderMsg", "*");
        defaultMQPushConsumer.setMessageModel(MessageModel.CLUSTERING);
        defaultMQPushConsumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {

                System.out.printf("%s 获取的消息 %s \n", LocalDateTime.now(), msgs);
				//返回消费不成功	
                return ConsumeConcurrentlyStatus.RECONSUME_LATER;
            }
        });
        defaultMQPushConsumer.start();

    }
}

```

> 上面例子是测试消息顺序



在消费端，消息消费返回的是非成功ConsumeConcurrentlyStatus.RECONSUME_LATER;

```java
public enum ConsumeConcurrentlyStatus {
    /**
     * Success consumption
     */
    CONSUME_SUCCESS,
    /**
     * Failure consumption,later try to consume
     */
    RECONSUME_LATER;
}

```



```
2022-05-26T15:02:58.780 获取的消息 [MessageExt [brokerName=broker-a, queueId=0, storeSize=170, queueOffset=10, sysFlag=0, bornTimestamp=1653548578737, bornHost=/172.16.5.115:55999, storeTimestamp=1653548486443, storeHost=/192.192.192.61:10911, msgId=C0C0C03D00002A9F00000000053A5FFD, commitLogOffset=87711741, bodyCRC=705268097, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='testOrderMsg', flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=11, CONSUME_START_TIME=1653548578780, UNIQ_KEY=AC100573068818B4AAC283F9AFB00000, WAIT=true}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 48], transactionId='null'}]] 
2022-05-26T15:03:09.182 获取的消息 [MessageExt [brokerName=broker-a, queueId=0, storeSize=306, queueOffset=0, sysFlag=0, bornTimestamp=1653548578737, bornHost=/172.16.5.115:55999, storeTimestamp=1653548496853, storeHost=/192.192.192.61:10911, msgId=C0C0C03D00002A9F00000000053A61D8, commitLogOffset=87712216, bodyCRC=705268097, reconsumeTimes=1, preparedTransactionOffset=0, toString()=Message{topic='testOrderMsg', flag=0, properties={MIN_OFFSET=0, REAL_TOPIC=%RETRY%ProducerGroup, ORIGIN_MESSAGE_ID=C0C0C03D00002A9F00000000053A5FFD, RETRY_TOPIC=testOrderMsg, MAX_OFFSET=1, CONSUME_START_TIME=1653548589182, UNIQ_KEY=AC100573068818B4AAC283F9AFB00000, WAIT=false, DELAY=3, REAL_QID=0}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 48], transactionId='null'}]] 
2022-05-26T15:03:39.189 获取的消息 [MessageExt [brokerName=broker-a, queueId=0, storeSize=306, queueOffset=1, sysFlag=0, bornTimestamp=1653548578737, bornHost=/172.16.5.115:55999, storeTimestamp=1653548526888, storeHost=/192.192.192.61:10911, msgId=C0C0C03D00002A9F00000000053A643B, commitLogOffset=87712827, bodyCRC=705268097, reconsumeTimes=2, preparedTransactionOffset=0, toString()=Message{topic='testOrderMsg', flag=0, properties={MIN_OFFSET=0, REAL_TOPIC=%RETRY%ProducerGroup, ORIGIN_MESSAGE_ID=C0C0C03D00002A9F00000000053A5FFD, RETRY_TOPIC=testOrderMsg, MAX_OFFSET=2, CONSUME_START_TIME=1653548619188, UNIQ_KEY=AC100573068818B4AAC283F9AFB00000, WAIT=false, DELAY=4, REAL_QID=0}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 48], transactionId='null'}]] 
2022-05-26T15:04:39.194 获取的消息 [MessageExt [brokerName=broker-a, queueId=0, storeSize=306, queueOffset=2, sysFlag=0, bornTimestamp=1653548578737, bornHost=/172.16.5.115:55999, storeTimestamp=1653548586892, storeHost=/192.192.192.61:10911, msgId=C0C0C03D00002A9F00000000053A669E, commitLogOffset=87713438, bodyCRC=705268097, reconsumeTimes=3, preparedTransactionOffset=0, toString()=Message{topic='testOrderMsg', flag=0, properties={MIN_OFFSET=0, REAL_TOPIC=%RETRY%ProducerGroup, ORIGIN_MESSAGE_ID=C0C0C03D00002A9F00000000053A5FFD, RETRY_TOPIC=testOrderMsg, MAX_OFFSET=3, CONSUME_START_TIME=1653548679194, UNIQ_KEY=AC100573068818B4AAC283F9AFB00000, WAIT=false, DELAY=5, REAL_QID=0}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 48], transactionId='null'}]] 
2022-05-26T15:06:39.202 获取的消息 [MessageExt [brokerName=broker-a, queueId=0, storeSize=306, queueOffset=3, sysFlag=0, bornTimestamp=1653548578737, bornHost=/172.16.5.115:55999, storeTimestamp=1653548706897, storeHost=/192.192.192.61:10911, msgId=C0C0C03D00002A9F00000000053A6901, commitLogOffset=87714049, bodyCRC=705268097, reconsumeTimes=4, preparedTransactionOffset=0, toString()=Message{topic='testOrderMsg', flag=0, properties={MIN_OFFSET=0, REAL_TOPIC=%RETRY%ProducerGroup, ORIGIN_MESSAGE_ID=C0C0C03D00002A9F00000000053A5FFD, RETRY_TOPIC=testOrderMsg, MAX_OFFSET=4, CONSUME_START_TIME=1653548799202, UNIQ_KEY=AC100573068818B4AAC283F9AFB00000, WAIT=false, DELAY=6, REAL_QID=0}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 48], transactionId='null'}]] 
2022-05-26T15:09:39.207 获取的消息 [MessageExt [brokerName=broker-a, queueId=0, storeSize=306, queueOffset=4, sysFlag=0, bornTimestamp=1653548578737, bornHost=/172.16.5.115:55999, storeTimestamp=1653548886902, storeHost=/192.192.192.61:10911, msgId=C0C0C03D00002A9F00000000053A6B64, commitLogOffset=87714660, bodyCRC=705268097, reconsumeTimes=5, preparedTransactionOffset=0, toString()=Message{topic='testOrderMsg', flag=0, properties={MIN_OFFSET=0, REAL_TOPIC=%RETRY%ProducerGroup, ORIGIN_MESSAGE_ID=C0C0C03D00002A9F00000000053A5FFD, RETRY_TOPIC=testOrderMsg, MAX_OFFSET=5, CONSUME_START_TIME=1653548979207, UNIQ_KEY=AC100573068818B4AAC283F9AFB00000, WAIT=false, DELAY=7, REAL_QID=0}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 48], transactionId='null'}]] 
2022-05-26T15:13:39.214 获取的消息 [MessageExt [brokerName=broker-a, queueId=0, storeSize=306, queueOffset=5, sysFlag=0, bornTimestamp=1653548578737, bornHost=/172.16.5.115:55999, storeTimestamp=1653549126905, storeHost=/192.192.192.61:10911, msgId=C0C0C03D00002A9F00000000053A6DC7, commitLogOffset=87715271, bodyCRC=705268097, reconsumeTimes=6, preparedTransactionOffset=0, toString()=Message{topic='testOrderMsg', flag=0, properties={MIN_OFFSET=0, REAL_TOPIC=%RETRY%ProducerGroup, ORIGIN_MESSAGE_ID=C0C0C03D00002A9F00000000053A5FFD, RETRY_TOPIC=testOrderMsg, MAX_OFFSET=6, CONSUME_START_TIME=1653549219214, UNIQ_KEY=AC100573068818B4AAC283F9AFB00000, WAIT=false, DELAY=8, REAL_QID=0}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 48], transactionId='null'}]] 
2022-05-26T15:18:39.224 获取的消息 [MessageExt [brokerName=broker-a, queueId=0, storeSize=306, queueOffset=6, sysFlag=0, bornTimestamp=1653548578737, bornHost=/172.16.5.115:55999, storeTimestamp=1653549426909, storeHost=/192.192.192.61:10911, msgId=C0C0C03D00002A9F00000000053A702A, commitLogOffset=87715882, bodyCRC=705268097, reconsumeTimes=7, preparedTransactionOffset=0, toString()=Message{topic='testOrderMsg', flag=0, properties={MIN_OFFSET=0, REAL_TOPIC=%RETRY%ProducerGroup, ORIGIN_MESSAGE_ID=C0C0C03D00002A9F00000000053A5FFD, RETRY_TOPIC=testOrderMsg, MAX_OFFSET=7, CONSUME_START_TIME=1653549519223, UNIQ_KEY=AC100573068818B4AAC283F9AFB00000, WAIT=false, DELAY=9, REAL_QID=0}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 48], transactionId='null'}]] 
2022-05-26T15:24:39.232 获取的消息 [MessageExt [brokerName=broker-a, queueId=0, storeSize=307, queueOffset=7, sysFlag=0, bornTimestamp=1653548578737, bornHost=/172.16.5.115:55999, storeTimestamp=1653549786912, storeHost=/192.192.192.61:10911, msgId=C0C0C03D00002A9F00000000053A728E, commitLogOffset=87716494, bodyCRC=705268097, reconsumeTimes=8, preparedTransactionOffset=0, toString()=Message{topic='testOrderMsg', flag=0, properties={MIN_OFFSET=0, REAL_TOPIC=%RETRY%ProducerGroup, ORIGIN_MESSAGE_ID=C0C0C03D00002A9F00000000053A5FFD, RETRY_TOPIC=testOrderMsg, MAX_OFFSET=8, CONSUME_START_TIME=1653549879232, UNIQ_KEY=AC100573068818B4AAC283F9AFB00000, WAIT=false, DELAY=10, REAL_QID=0}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 48], transactionId='null'}]] 
2022-05-26T15:31:39.244 获取的消息 [MessageExt [brokerName=broker-a, queueId=0, storeSize=307, queueOffset=8, sysFlag=0, bornTimestamp=1653548578737, bornHost=/172.16.5.115:55999, storeTimestamp=1653550206916, storeHost=/192.192.192.61:10911, msgId=C0C0C03D00002A9F00000000053A74F3, commitLogOffset=87717107, bodyCRC=705268097, reconsumeTimes=9, preparedTransactionOffset=0, toString()=Message{topic='testOrderMsg', flag=0, properties={MIN_OFFSET=0, REAL_TOPIC=%RETRY%ProducerGroup, ORIGIN_MESSAGE_ID=C0C0C03D00002A9F00000000053A5FFD, RETRY_TOPIC=testOrderMsg, MAX_OFFSET=9, CONSUME_START_TIME=1653550299244, UNIQ_KEY=AC100573068818B4AAC283F9AFB00000, WAIT=false, DELAY=11, REAL_QID=0}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 48], transactionId='null'}]] 

```

通过关键字“获取的消息”，发现已经重试了10次了。这里的重试逻辑是如下

消费端消息没有消费成功，RocketMQ会放到重试队列。重试topic的名字是%RETRY%+consumergroup名字（这里需要注意的是，这个Topic的重试队列是针对消费组，而不是针对每个Topic设置的）;上面的消费组名称是ProducerGroup，那么对应的重试队列是%RETRY%ProducerGroup； 用于暂时保存因为各种异常而导致Consumer端无法消费的消息。考虑到异常恢复起来需要一些时间，会为重试队列设置多个重试级别，每个重试级别都有与之对应的重新投递延时，重试次数越多投递延时就越大。RocketMQ对于重试消息的处理是先保存至Topic名称为“SCHEDULE_TOPIC_XXXX”的延迟队列中，后台定时任务按照对应的时间进行Delay后重新保存至“%RETRY%+consumerGroup”的重试队列中。如果重复消费都失败并且到达一定次数（默认16次），就会投递到DLQ死信队列；





![封面](P1.png)

​	重试队列

![封面](P2.png)

​	队列配置



### 消息重试

Consumer消费消息失败后，要提供一种重试机制，让消息再消费一次。Consumer消费消息失败通常可以认为有以下几种情况：

- 由于消息本身的原因，例如反序列化失败、消息数据本身无法处理（例如话费充值，当前消息的手机号被注销，无法充值）等。这种错误通常需要跳过这条消息，再消费其它消息，而这条失败的消息即使立刻重试消费，99%也不成功，所以最好提供一种定时重试机制，即过10秒后再重试。
- 由于依赖的下游应用服务不可用，例如db连接不可用，外系统网络不可达等。遇到这种错误，即使跳过当前失败的消息，消费其他消息同样也会报错。这种情况建议应用sleep 30s，再消费下一条消息，这样可以减轻Broker重试消息的压力。



参考资料：

​	RocketMQ官网





