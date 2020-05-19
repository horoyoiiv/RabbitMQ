

# Direct  

#### 상황  
* 하나의 `소스 어플리케이션`이 있고, 그곳의 `로그`를 처리하려고 한다.  
* 하나의 `타킷 어플리케이션`에서는 모든 종류의 로그를 화면에 띄어준다.  
* 다른 하나의 `타깃 어플리케이션`에서는 `ERROR` 로그만 Disk에 저장한다.  


## 1. 바인딩 키  

* **Exchange**에 **메세지 큐**를 붙일 때 사용.  
* 해당 `바인딩 키`를 통해 메세지가 **라우팅**된다.  

## 2. 바인딩 키 설정  
```java
channel.queueBind(queueName, EXCHANGE_NAME, "black");
```
* 생성된 큐를 익스체인지에 붙일 때, 세번째 인자로 black을 넣어준다.  

![스크린샷, 2020-05-19 22-38-51](https://user-images.githubusercontent.com/62331555/82333314-8984c700-9a21-11ea-9d18-bccd27f0f451.png)  

## 3. 라우팅 키  
* 생산자에서 생성한 `라우팅 키`와 `바인딩 키`가 일치하는 `메세지 큐`로 메세지를 전달.  

* 하나의 `Exchange`에 동일한 `바인딩 키`를 가지는 `메시지 큐`가 당연히 붙어도 된다.  
![스크린샷, 2020-05-19 22-38-51](https://user-images.githubusercontent.com/62331555/82333641-f4ce9900-9a21-11ea-83b8-66da43b22583.png)  


## 4. 생산자  

```java

        try(    // try-with-resources
            Connection con = factory.newConnection();
            Channel channel = con.createChannel();
        ){
            channel.exchangeDeclare(EXCHANGE_NAME, "direct");

            String severity = argv[0];
            String message = argv[1];

            channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + severity + "':'" + message + "'");
             
```

* 해당 `Exchange`에 `severity변수 값`이라는 **라우팅 키**를 통해 메세지를 전달.   
```java
channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes("UTF-8"));  
```

## 5 소비자 
```java
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        String queueName = channel.queueDeclare().getQueue();

        if (argv.length < 1) {
            System.err.println("Usage: ReceiveLogsDirect [info] [warning] [error]");
            System.exit(1);
        }

        for (String severity : argv) {
            channel.queueBind(queueName, EXCHANGE_NAME, severity);
        }

        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" + delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };
```

* 생성된 `큐`를 `Exchange`에 바인딩 시키고 이 때, `바인딩 키`를 사용하여 붙인다.  

```java
        for (String severity : argv) {
            channel.queueBind(queueName, EXCHANGE_NAME, severity);
        }
```

#### 
![스크린샷, 2020-05-19 23-14-11](https://user-images.githubusercontent.com/62331555/82337432-b7203f00-9a26-11ea-9e89-a9ff2581c21f.png)  








