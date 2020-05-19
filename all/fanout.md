

# Exchage  

* Exchage를 사용할 때가 왔다.  

* RabbitMQ에서 `생산자`는 직접적으로 `메세지 큐`에 메세지를 전달하는 것이 아닌, `Exchange`를 경유하여 전달한다.  

* `Exchange`의 종류에는 `fanout`, `direct`, `topic`, `header`가 있다.  

## 1. fanout 방식  

* `fanout`이 의미하듯이, Exchange에 연결된 모든 `메세지 큐`에 메세지를 전달한다.  

**Note**  
* 앞선 예제에서 `Exchange`를 "" 빈 문자열로 설정했다.  
* 이는 `default exchange`로 가게 된다.  
```java
channel.basicPublish("", "hello", null, message.getBytes());
```

![스크린샷, 2020-05-19 22-08-33](https://user-images.githubusercontent.com/62331555/82330069-4de7fe00-9a1d-11ea-87c8-5c595ab5fb0e.png)  

## 2. 생산자  


```java

public class App {
    private final static String EXCHANGE_NAME = "logs";

    public String getGreeting() {
        return "Hello world.";
    }

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
            channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
  
            String message = "Hello World";

            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + message + "'");
        
        }catch (TimeoutException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } 

    }
}
```

#### exchangeDeclare  
* `Exchange`를 생성한다.  
* `Exchange Type`을 **fanout**으로 지정한다.  
```java
private final static String EXCHANGE_NAME = "logs";
channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
```

#### basicPublish  
* 두번째 인자인 **라우팅 키**는 fanout에서 생략.  
```java
channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
```

## 3 소비자  

```java
  public static void main(String[] argv) throws Exception {
    
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    // 익스체인지 생성
    channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
    
    // 큐 생성
    String queueName = channel.queueDeclare().getQueue();
    
    // 생성된 큐를 익스테인지에 바인딩
    channel.queueBind(queueName, EXCHANGE_NAME, "");

    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    DeliverCallback deliverCallback = (consumerTag, delivery) -> {
        String message = new String(delivery.getBody(), "UTF-8");
        System.out.println(" [x] Received '" + message + "'");
    };
    
    channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
  }
```

#### channel.queueDeclare().getQueue();  
* RabbitMQ 서버가 알아서 큐를 생성시키는 API  
* 두 개의 소비자 어플리케이션에서 이를 호출하면 아래와 같이 두 개의 큐가 생성된다.  

#### channel.queueBind(queueName, "logs", "");  
* 생성한 `메세지 큐`를 `Exchange`에 **바인딩**시킨다.  

![스크린샷, 2020-05-19 22-23-12](https://user-images.githubusercontent.com/62331555/82331919-b637df00-9a1f-11ea-9e20-6522d57c48ed.png)  

## 결과  

* 이를 통해, `생산자`에서 전송된 `메세지`는 `Exchange`를 거쳐, `Exchange`에 바인딩된 모든 `메세지 큐`에 **브로드캐스팅**된다.  




  










