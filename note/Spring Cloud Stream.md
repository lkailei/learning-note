# Spring Cloud Stream

## 基本概念

在实际的开发过程中，服务与服务之间经常会使用到消息中间件，用于解决应用解耦 ，异步处理，流量削峰等问题实现高性能，高可用，最终一致性的架构。

Spring Cloud Stream是一个用来为微服务应用构建消息驱动能力的框架。它可以基于Spring Boot来创建独立的、可用于生产的Spring应用程序。它通过使用Spring Integration来连接消息代理中间件以实现消息事件驱动的微服务应用。使用Stream可以降低中间件与我们系统之间的 耦合性，不如我们可以让rabbitMq切换为Kafka,可以不用修改代码。只需要修改一些配置即可。

- 独立地构建、测试和部署以数据为中心的应用程序
- 应用现代微服务体系结构模式，包括通过消息传递进行组合
- 使用以事件为中心的思想来分离应用程序职责。事件可以表示在时间上已经发生的事情，下游消费者应用程序可以对此作出反应，而不必知道事件的起源或生产者的身份。
- 将业务逻辑移植到消息代理上(比如 RabbitMQ、 Apache Kafka、 Amazon kinesis)。
- 依赖于框架对常见用例的自动内容类型支持。扩展到不同的数据转换类型是可能的。

## 消息中间件应用场景

### 应用解耦

假设有上百个系统都需要系统A的核心数据，此时负责系统A的工程师将是崩溃的，一旦有系统加入，A系统就需要修改代码，将数据发送到新加入的系统。反之，如果有系统不再需要A发送数据，那么A系统又得修改代码不再向其发送数据。这样的架构设计耦合度太高了，我们就可以引入消息中间件来实现系统之间的解耦。即核心系统A生产核心数据，然后将核心数据发送到消息中间件，下游消费系统根据自身需求从中间件里获取消息进行消费，当不再需要数据时就不取消息进即可，这样系统之间耦合度就大大降低了。具体流程图如下。

[![48Ysx0.png](https://z3.ax1x.com/2021/09/19/48Ysx0.png)](https://imgtu.com/i/48Ysx0)

### 异步处理

比如在电商网站用户下单，下单完成后会给用户进行发送短信或者邮件，而这个发送短信和邮件过程不属于核心业务所以我们可以将其放到其他服务中，我们将原来的流式形式改为中间件模式

[![48tokQ.png](https://z3.ax1x.com/2021/09/19/48tokQ.png)](https://imgtu.com/i/48tokQ)

### 流量削峰

比如秒杀服务，一下子进入许多需求针对这个场景我们可以将其中间加入一层消息队列，将请求的先入队列，然后再把队列中的请求平滑的推送给服务，或者让服务去队列中进行拉去。

### 日志处理

对于小型项目来说，我们通常对日志的处理没有那么多要求，但当用户量达到一定的峰值后，问题便会出现。这式可以用Kafka进行对日志的的处理。

## 核心概念

[![48d1j1.png](https://z3.ax1x.com/2021/09/19/48d1j1.png)](https://imgtu.com/i/48d1j1)

| 组成            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| Middleware      | 中间件支持RabbitMQ和Kafka                                    |
| Binder          | 目标绑定器，目标指的是Kafka或者RabbitMq，绑定器就是封装了目标中间件的包，如果操作的是Kafka就是使用spring-cloud-stream-binder-kafka,如果操作是rabbitMq就是使用spring-sttream-binder-rabbit |
| @input          | 注解标识 输入通道，接收（消息消费者）的消息，将通过该通道进入应用程序。 |
| @output         | 注解标识输出通道，发布（消息产生者）的消息，将通过该通道离开应用程序 |
| @StreamListener | 监听队列，消费者的队列的消息接收 和@Input一起使用            |
| @EnableBinding  | 注解标识绑定，将信道channel和交换机exchange绑定一起          |

## 工作原理

通过定义绑定器作为中间层，实现应用程序与消息中间件细节的隔离，通过项应用程序暴漏统一的channel通道，使得应用程序不再考虑各种的消息中间件的实现，当需要升级消息中间件，或者使用更换其他的中间的产品时，我们需要做的就是更换对应的Binder绑定器而不需要修改其他的逻辑

[![4GCjMV.png](https://z3.ax1x.com/2021/09/20/4GCjMV.png)](https://imgtu.com/i/4GCjMV)

- Source: 当需要发送消息时，我们就 可以通过Source.java，他会 把我们所需要发送的消息进行序列化，然后将这些书籍发送到channel中
- Sink：放我们需要监听消息时，就需要通过Sink.java，它负责从消息通道中获取消息，并将消息反序列化成消息对应的格式，然后交给具体的 监听处理
- Channel ：通常我们向消息中间件发送消息或者 监听  消息时需要指定主题（topic）和消息队列名称，一旦我们需要变更主题的时候就需要修改消息发送或者消息监听的代码，通过channel对象配置哪个主题，这样变更主题时候就不同更改代码，从而实现中间 件的解耦
- Binder：通过不同的Binder可以实现与不同的消息中间件整合，Binder提供统一的消息收发接口，从而使得我们可以根据实际需要部署不同的消息中间件，或者根据实际生产中所部署的消息中间件来调整我们的配置。

## 实例

### **依赖**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

### 生产者

```yaml
spring:
  rabbitmq:
    host: 192.168.126.128
    port: 5672
    username: rabbit
    password: 123456
    virtual-host:
  cloud:
  	stream:
  	  bindings:
  	  	# 消息发送通道，同样也可以自定义配置通道
		output:
		  destination: stream.message # 绑定的交换机的名称
		# 自定义通道的名称 
		authorityOutput: 
		  destination: stream.authority # 绑定的交换机的名称
```

自定义配置发送通道接口

```java
public interface AuthoritySource{
	String AUHORITY_OUTPUT = "authorityOutput";
	
	@Output(AuthoritySource.AUTHORITY_OUTPUT)
	MessageChannel authorityOutput();
}
```

将信道与exchange绑定在一起，进行发送消息到channel中

```java
/**
* 消息生产者
**/
@EnableBinding(AuthoritySource.class)
public class AuthorityProduce {
    // 注入自定义通道
	@Autowired
	private AuhoritySource authoritySource;
    
    // 注入默认通道
    @Autowired
    private Source source;
    
	/**
	* 使用自定义通道发送消息
	**/
	public void send(String message){
		authoritySource.output().send(MessageBuilder.withPayload(message).build())
	}
	/**
	* 使用默认通道发送消息
	**/
    public void send(String message){
        source.output().send(Mes看   sageBuilder.withPayload(message).build())
    }
}
```

### 消费者

```
spring:
  rabbitmq:
    host: 192.168.126.128
    port: 5672
    username: rabbit
    password: 123456
    virtual-host:
  cloud:
  	stream:
  	  bindings:
  	  	# 消息发送通道，同样也可以自定义配置通道
		input:
		  destination: stream.message # 绑定的交换机的名称
		# 自定义通道的名称 
		authorityInput: 
		  destination: stream.authority # 绑定的交换机的名称
```

自定义配置发送通道接口

```java
public interface AuthoritySink{
	String AUHORITY_INPUT = "authorityInput";
	
	@Output(AuthoritySink.AUTHORITY_INPUT)
	SubScribableChannel authorityInput();
}
```

将消息和信道交换器进行绑定

```java
/**
*
* 自定义消息消费者
**/
@EnableBinding(AuthoritySink.class)
public class MessageConsumer{
    /**
    * 接收消息
    **/
    @StreamListener(AuthoritySink.AUHORITY_INPUT)
    public void receive(String message){
        System.out.println("message="+message);
    }
}
/**
*
* 默认消息消费者
**/
@EnableBinding(Sink.class)
public class MessageConsumer{
    /**
    * 接收消息
    **/
    @StreamListener(Sink.INPUT)
    public void receive(String message){
        System.out.println("message="+message);
    }
}
```

## 消息分组



## 消息分区

​                                                                                      

消息消费的保证

1. 如果消息未成功消费(消费失败)可通过以下解决。

1. 1. 将消息重新塞入队列 。通过配置`spring.cloud.stream.bindings.example-topic-input.consumer.requeue-rejected=true` 这个配置会将消费失败的消息进行重新执行。直到消费成功为止。
   2. 自动重试。设置重试次数 spring cloud stream默认的会设置重试次数为3次 同时我们可以进行修改重试次数的默认值 `spring.cloud.stream.bindings.example-topic-input.consumer.max-attempts=1`。如果超过重试次数则进入死信对列进入DLQ进行处理
   3. 进行降级处理。通过在消费端进行配置` @ServiceActivator(inputchannel="test-topic.stream-exception-handler.errors") `注解的方法进行执行。`test-topic->destination . handler-> group `相应的降级的。这一种也不能真正保证 也只能通过记录和后续处理来解决 
   4. 使用DLQ队列。开启DLQ `spring.cloud.stream.rabbit.bindings.example-topic-input.consumer.auto-bind-dlq=true` （这个配置+设置计数器 能够解决消息堆积问题）。 需要装DLQ插件 可以控制消息在DLQ中存活时间。还可以配置消息加入死信队列的时候的错误。

如何保证消息不会被重复消费

1. 将消息进行分组。当我们指定了某个绑定所指向的消费组之后，往当前主题发送的消息在每个订阅消费组中，只会有一个订阅者接收和消费，从而实现了对消息的负载均衡。只所以之前会出现重复消费的问题，是由于默认情况下，任何订阅都会产生一个匿名消费组，所以每个订阅实例都会有自己的消费组，从而当有消息发送的时候，就形成了广播的模式.只需要在生产者和消费者加入group属性即可。

## ChannelInterceptor

ChannelInterceptor 拦截器拦截 ws 协议的请求/响应(`send/subscribe`)，不拦截 http 协议的请求/响应。

[spring cloud stream channel intercaptor](https://dzone.com/articles/spring-cloud-stream-channel-interceptor)

```
//方法调用顺序：preSend -> postSend -> afterSendCompletion
public class SocketChannelInterceptor implements ChannelInterceptor {

    /**
     * 消息发送到目标方法之前被调用
     *
     * @param message
     * @param channel
     * @return
     */
    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
        System.out.println("preSend->");
        return ChannelInterceptor.super.preSend(message, channel);
    }

    /**
     * 消息正在发送过程中被调用，主要用这个方法就行
     *
     * @param message
     * @param channel
     * @param sent
     */
    @Override
    public void postSend(Message<?> message, MessageChannel channel, boolean sent) {
        StompHeaderAccessor header = StompHeaderAccessor.wrap(message);
        System.out.println("postSend->" + header.getDestination());
    }

    /**
     * 消息发送完成之后被调用，一般用来做资源释放等工作。
     *
     * @param message
     * @param channel
     * @param sent
     * @param ex
     */
    @Override
    public void afterSendCompletion(Message<?> message, MessageChannel channel, boolean sent, Exception ex) {
        System.out.println("afterSendCompletion->");
        ChannelInterceptor.super.afterSendCompletion(message, channel, sent, ex);
    }
}

```

2. 配置中进行添加拦截器

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new SocketChannelInterceptor());
    }

    @Override
    public void configureClientOutboundChannel(ChannelRegistration registration) {
        registration.interceptors(new SocketChannelInterceptor());
    }
}
```

