---
title: RocketMQ保证消息顺序
date: 2022-05-26 15:57:10
tags:
---



### 顺序消息

消息有序指的是一类消息消费时，能按照发送的顺序来消费。例如：一个订单产生了三条消息分别是订单创建、订单付款、订单完成。消费时要按照这个顺序消费才能有意义，但是同时订单之间是可以并行消费的。RocketMQ可以严格的保证消息有序。

顺序消息分为全局顺序消息与分区顺序消息，全局顺序是指某个Topic下的所有消息都要保证顺序；部分顺序消息只要保证每一组消息被顺序消费即可。

- 全局顺序
  对于指定的一个 Topic，所有消息按照严格的先入先出（FIFO）的顺序进行发布和消费。
  适用场景：性能要求不高，所有的消息严格按照 FIFO 原则进行消息发布和消费的场景

- 分区顺序
  对于指定的一个 Topic，所有消息根据 **sharding key**进行区块分区。 同一个分区内的消息按照严格的 FIFO 顺序进行发布和消费。 Sharding key 是顺序消息中用来区分不同分区的关键字段，和普通消息的 Key 是完全不同的概念。
  适用场景：性能要求高，以 sharding key 作为分区字段，在同一个区块中严格的按照 FIFO 原则进行消息发布和消费的场景。

  

> from 官网，实际使用中用分区顺序



### 源码演示

消息主题testOrderMsg ，一共配置了10个队列

![封面](P1.png)

**Producer**

```java
public class OrderMessage {


    public static void main(String[] args) throws MQBrokerException, RemotingException, InterruptedException, MQClientException {

        TransactionMQProducer producer = new TransactionMQProducer("testOrderMsgGroup");

        producer.setNamesrvAddr("192.192.192.61:9876");
        producer.start();
        MsgSelector msgSelector = new MsgSelector();

        for (int i = 0; i < 100; i++) {
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

```java
public class OrderMsgConsumer {

    public static void main(String[] args) throws MQClientException {

        DefaultMQPushConsumer defaultMQPushConsumer = new DefaultMQPushConsumer("ProducerGroup");

        defaultMQPushConsumer.setNamesrvAddr("192.192.192.61:9876");
        defaultMQPushConsumer.subscribe("testOrderMsg", "*");
        defaultMQPushConsumer.setMessageModel(MessageModel.CLUSTERING);

        //  defaultMQPushConsumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

        defaultMQPushConsumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {

                System.out.printf("%s 获取的消息 %s \n", LocalDateTime.now(), msgs);

                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        defaultMQPushConsumer.start();
        
    }
}
```



MsgSelector类就用来做消息路由，通过arg参数和消息队列个数取模运算；关键代码如下:

```java
 public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
            Integer id = (Integer) arg;
            int index = id % mqs.size();
            return mqs.get(index);
        }
```
控制台查看消费情况，每个队列10条消息，这样实现了消息的顺序性；

![封面](P3.png)



### SpringBoot发送顺序消息方式

在SpringBoot中，发送消息是集成在RocketMQTemplate类中；是对DefaultMQProducer的封装，发送顺序消息由两个方法

同步方式：

```java
public SendResult syncSendOrderly(String destination, Message<?> message, String hashKey) {
    return syncSendOrderly(destination, message, hashKey, producer.getSendMsgTimeout());
}
```

异步方式：

```java
public void asyncSendOrderly(String destination, Message<?> message, String hashKey, SendCallback sendCallback,
                             long timeout) {
    if (Objects.isNull(message) || Objects.isNull(message.getPayload())) {
        log.error("asyncSendOrderly failed. destination:{}, message is null ", destination);
        throw new IllegalArgumentException("`message` and `message.payload` cannot be null");
    }

    try {
        org.apache.rocketmq.common.message.Message rocketMsg = RocketMQUtil.convertToRocketMessage(objectMapper,
            charset, destination, message);
        producer.send(rocketMsg, messageQueueSelector, hashKey, sendCallback, timeout);
    } catch (Exception e) {
        log.error("asyncSendOrderly failed. destination:{}, message:{} ", destination, message);
        throw new MessagingException(e.getMessage(), e);
    }
}
```



这里面使用了messageQueueSelector变量。默认是SelectMessageQueueByHash。

```java
private MessageQueueSelector messageQueueSelector = new SelectMessageQueueByHash();
```



SelectMessageQueueByHash 源码如下：

```java
public class SelectMessageQueueByHash implements MessageQueueSelector {

    @Override
    public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
        int value = arg.hashCode();
        if (value < 0) {
            value = Math.abs(value);
        }
        
		//value和消息队列长度取模
        value = value % mqs.size();
        return mqs.get(value);
    }
}
```



### 消费端处理

上面都是通过生产者方式保证了顺序消费，但是消费端没做处理，这是由于消费机制保证。消费端只有本条消息消费了，才能获取下一条消息，消费端个数不受影响；





