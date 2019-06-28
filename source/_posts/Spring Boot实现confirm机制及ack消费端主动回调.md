---
title: RabbitMQ数据丢失问题，Spring Boot实现confirm机制及ack消费端主动回调
---
#### 1:说明：ACK是默认是自动，在消息发送给消费者后立即确认。所以若消费端消费业务逻辑抛出异常，会可能丢失消息。即便加入事务回滚了也只保证数据的一致性，而消息依然丢失。所以，若消费端未成功处理此条消息，消息就会丢失。
  NONE（默认）：自动；AUTO：根据情况确认；MANUAL：手动确认

#### 2:yml配置：
        
     spring: rabbitmq: host: 127.0.0.1 
     port: 5672 
     username: guest 
     password: guest 
     #发送确认 对应RabbitTemplate.ConfirmCallback接口 
     publisher-confirms: true 
     #发送失败回退，对应RabbitTemplate.ReturnCallback接口 
     publisher-returns: true 
     #手动提交ack 
     listener: 
        direct: 
            #NONE（默认）：自动；AUTO：根据情况确认；MANUAL：手动确认 
            acknowledge-mode: manual 
        simple: 
            acknowledge-mode: manual
            
#### 3:config配置：
    
    
    @Configuration public class RabbitConfig { /**
         * 定义一个队列
         * @return
         */ 
         @Bean 
         public Queue okongQueue() { 
            return new Queue("user"); 
         } 
         @Bean 
         MessageConverter messageConverter() {
            return new Jackson2JsonMessageConverter(); 
         }
     }
     
#### 4:发送端：

    @Service
    public class SenderService implements RabbitTemplate.ConfirmCallback,
        RabbitTemplate.ReturnCallback {
        @Autowired
        private RabbitTemplate rabbitTemplate;
    
        public void sendMq() throws InterruptedException {
            this.rabbitTemplate.setConfirmCallback(this);
            rabbitTemplate.convertAndSend("user", "6666666");
        }
    
        @Override
        public void confirm(CorrelationData correlationData, boolean ack,
            String cause) {
            System.out.println("=====已消费======");
    
            if (ack) {
                System.out.println("消息: " + correlationData + "，已经被ack成功");
            } else {
                System.out.println("消息: " + correlationData + "，nack，失败原因是：" +
                    cause);
            }
        }
    
        @Override
        public void returnedMessage(Message message, int replyCode,
            String replyText, String exchange, String routingKey) {
            System.out.println("sender return success" + message.toString());
        }
    }
#### 5：消费端

    @Service
    public class SenderService implements RabbitTemplate.ConfirmCallback,
        RabbitTemplate.ReturnCallback {
        @Autowired
        private RabbitTemplate rabbitTemplate;
    
        public void sendMq() throws InterruptedException {
            this.rabbitTemplate.setConfirmCallback(this);
            rabbitTemplate.convertAndSend("user", "6666666");
        }
    
        @Override
        public void confirm(CorrelationData correlationData, boolean ack,
            String cause) {
            System.out.println("=====已消费======");
    
            if (ack) {
                System.out.println("消息: " + correlationData + "，已经被ack成功");
            } else {
                System.out.println("消息: " + correlationData + "，nack，失败原因是：" +
                    cause);
            }
        }
    
        @Override
        public void returnedMessage(Message message, int replyCode,
            String replyText, String exchange, String routingKey) {
            System.out.println("sender return success" + message.toString());
        }
    }

还可指定以下处理方式，可参考api：

    @Service
    public class SenderService implements RabbitTemplate.ConfirmCallback,
        RabbitTemplate.ReturnCallback {
        @Autowired
        private RabbitTemplate rabbitTemplate;
    
        public void sendMq() throws InterruptedException {
            this.rabbitTemplate.setConfirmCallback(this);
            rabbitTemplate.convertAndSend("user", "6666666");
        }
    
        @Override
        public void confirm(CorrelationData correlationData, boolean ack,
            String cause) {
            System.out.println("=====已消费======");
    
            if (ack) {
                System.out.println("消息: " + correlationData + "，已经被ack成功");
            } else {
                System.out.println("消息: " + correlationData + "，nack，失败原因是：" +
                    cause);
            }
        }
    
        @Override
        public void returnedMessage(Message message, int replyCode,
            String replyText, String exchange, String routingKey) {
            System.out.println("sender return success" + message.toString());
        }
    }

参数说明：

basicAck 方法需要传递两个参数
deliveryTag（唯一标识 ID）：当一个消费者向 RabbitMQ 注册后，会建立起一个 Channel ，RabbitMQ 会用 basic.deliver 方法向消费者推送消息，这个方法携带了一个 delivery tag， 它代表了 RabbitMQ 向该 Channel 投递的这条消息的唯一标识 ID，是一个单调递增的正整数，delivery tag 的范围仅限于 Channel
multiple：为了减少网络流量，手动确认可以被批处理，当该参数为 true 时，则可以一次性确认 delivery_tag 小于等于传入值的所有消息

basicNack方法需要传递三个参数
deliveryTag（唯一标识 ID）：同上
multiple：同上
requeue： true ：重回队列，false ：丢弃，我们在nack方法中必须设置 false，否则重发没有意义。

basicReject方法需要传递两个参数
deliveryTag（唯一标识 ID）：同上
requeue：同上

#### 6：指定控制层调用：

    @Service
    public class SenderService implements RabbitTemplate.ConfirmCallback,
        RabbitTemplate.ReturnCallback {
        @Autowired
        private RabbitTemplate rabbitTemplate;
    
        public void sendMq() throws InterruptedException {
            this.rabbitTemplate.setConfirmCallback(this);
            rabbitTemplate.convertAndSend("user", "6666666");
        }
    
        @Override
        public void confirm(CorrelationData correlationData, boolean ack,
            String cause) {
            System.out.println("=====已消费======");
    
            if (ack) {
                System.out.println("消息: " + correlationData + "，已经被ack成功");
            } else {
                System.out.println("消息: " + correlationData + "，nack，失败原因是：" +
                    cause);
            }
        }
    
        @Override
        public void returnedMessage(Message message, int replyCode,
            String replyText, String exchange, String routingKey) {
            System.out.println("sender return success" + message.toString());
        }
    }


