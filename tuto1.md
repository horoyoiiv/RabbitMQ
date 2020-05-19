

# Worker Queue  

![스크린샷, 2020-05-19 20-18-59](https://user-images.githubusercontent.com/62331555/82320367-19207a80-9a0e-11ea-9f9c-9492fcb11cdf.png)  

## 1. 생산자  


```java

public class App {
    private final static String QNAME = "task_queue";

    public String getGreeting() {
        return "Hello world.";
    }

    public static void main(String[] args) {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost"); 
        factory.setPort(5672);
        factory.setUsername("student");
        factory.setPassword("student");
        factory.setVirtualHost("/");

        try(    // try-with-resources
            Connection con = factory.newConnection();
            Channel channel = con.createChannel();
        ){
            channel.queueDeclare(QNAME, true, false, false, null);

            String message = "Hello World!";
            
            channel.basicPublish("", QNAME, null, message.getBytes());
            
            System.out.println(" [x] Sent '" + message + "'");
           
        }catch (TimeoutException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } 

    }
}
```
#### 1.1. queueDeclare  
* 브로커에 큐를 생성한다.  
```java
AMQP.Queue.DeclareOk queueDeclare(java.lang.String queue,
                                  boolean passive,
                                  boolean durable,
                                  boolean exclusive,
                                  boolean autoDelete,
                                  java.util.Map<java.lang.String,java.lang.Object> arguments)
                                  throws java.io.IOException
```
```java
channel.queueDeclare(QNAME, true, false, false, null);
```
* **passive** : true라면 이미 존재하는 경우 그걸 그대로 사용.  
* **durable** : 서버가 다운된 후 재시작 시 해당 큐를 그대로 살릴 수 있다.  

#### 1.2. basiPublish  
```java
void basicPublish(java.lang.String exchange,
                  java.lang.String routingKey,
                  AMQP.BasicProperties props,
                  byte[] body)
                  throws java.io.IOException
```

```java
channel.basicPublish("", QNAME, null, message.getBytes());
```
* **exchange** : 메세지를 전송할 `exchange` 설정  
* **routingKey** : 라우팅 키  



## 2. 소비자  

```java
    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        channel.basicQos(1);

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");

            System.out.println(" [x] Received '" + message + "'");
            try {
                doWork(message);
            } finally {
                System.out.println(" [x] Done");
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };
        
        
        boolean autoAck = false;
        channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, consumerTag -> { });
    }
```

## 3. 라운드 로빈 방식의 디스패칭  
* 하나의 큐에 여러 소비자가 붙어있다면 기본적으로 **라운드 로빈** 방식으로 메세지를 분산한다.  
* 이는 곧 병렬 처리가 용이함을 의미하고 **확장성**이 좋다는 것을 의미한다.  

## 4. 메세지 어크놀로지먼트  
* 만일 소비자가 메세지를 가져갔는데, 처리 도중 다운되어버리면 그 메세지를 처리하지 못 하게 된다.  
* `브로커`로 하여금 `소비자`가 자신이 다 처리했음을 의미하는 `ACK` 패킷을 보낼 때 비로소 `메세지 큐`에서 삭제하라는 신호를 보낼 수 있다.  
* 이를 통해 `메세지의 손실`을 방지할 수 있다.  

```java
boolean autoAck = false;
channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, consumerTag -> { });
```

## 5. 브로커에서의 메세지 손실 방지  

* 생산 시 durable=true를 통하여 해당 메세지를 `디스크`에 저장하게 한다.  
```java
boolean durable = true;
channel.queueDeclare("task_queue", durable, false, false, null);
```








