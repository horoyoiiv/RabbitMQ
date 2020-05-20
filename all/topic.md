

# topic 기반의 라우팅  

* `direct` 방식은 `하나의 문자열`을 기준으로 라우팅을 수행했다.  
* **topic** 기반의 `Exchange`는 여러 문자열로 구성된 `topic`을 기준으로 라우팅을 수행한다.  

![스크린샷, 2020-05-21 01-11-15](https://user-images.githubusercontent.com/62331555/82470216-05554100-9b00-11ea-8b9f-09517a973318.png)  

## 1. wildcard  

* `*` : (star) single level wildcard - 하나의 단어를 대체한다.  
* `#` : (hash) multi level wildcard  - 여러 단어를 대체한다.  


## 2. 생산자  
* `라우팅 키`에 `토픽 스트링`을 넣어서 전송한다.  

```java
    public static void main(String[] argv) {
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
            channel.exchangeDeclare(EXCHANGE_NAME, "topic");

            String topics = argv[0];
            String message = argv[1];

            channel.basicPublish(EXCHANGE_NAME, topics, null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + topics + "':'" + message + "'");
             
        }catch (TimeoutException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } 
    }
```

## 3. 소비자  
* `Exchange` 타입을 `Topic`으로 설정한다.  
* `토픽 패턴`을 `바인딩 키`로 설정한다.  


```java
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
        String queueName = channel.queueDeclare().getQueue();
        
        channel.queueBind(queueName, EXCHANGE_NAME, argv[0]);
        
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" + delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };

        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
```

