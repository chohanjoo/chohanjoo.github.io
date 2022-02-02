---
title: "spring-boot RabbitMQ 테스트 코드"
description: rabbitmq-mock 을 사용하여 AMQP 를 테스트한다.
categories:
 - rabbitmq
tags:
- rabbitmq
- amqp
---

# rabbitmq-mock

rabbitmq의 테스트 코드를 작성하기 위해 `rabbitmq-mock` 프로젝트를 이용하려고 한다.

[github](https://github.com/fridujo/rabbitmq-mock) 을 통해 프로젝트를 볼 수 았다.

<br/>

## Dependency

```gradle
testImplementation('com.github.fridujo:rabbitmq-mock:1.1.1')
```

<br/>

## Test Code 

해당 프로젝트의 test code 를 가져왔다.


### Bean 추가 

```java
@TestConfiguration
public class AmqpTestConfig {

    @Bean
    ConnectionFactory connectionFactory() {
        return new CachingConnectionFactory(
                MockConnectionFactoryFactory
                        .build()
                        .enableConsistentHashPlugin()
        );
    }

    @Bean
    RabbitAdmin rabbitAdmin(ConnectionFactory connectionFactory) {
        return new RabbitAdmin(connectionFactory);
    }

    @Bean
    RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setMessageConverter(messageConverter());
        return rabbitTemplate;
    }

    @Bean
    RabbitTemplate testRabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setMessageConverter(messageConverter());
        return rabbitTemplate;
    }

    @Bean
    MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

<br/>

### Direct Exchange Test Code

```java
    @Test
    public void helloWorld() {
        Msg msg = Msg.builder()
                .content("hi")
                .build();

        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AmqpTestConfig.class);

        RabbitTemplate rabbitTemplate = queueAndExchangeSetup(context);
        rabbitTemplate.convertAndSend(CodeConstants.AMQP.MSG_EXCHANGE, CodeConstants.AMQP.ROUTING_KEY, msg.toString());

        Msg message = rabbitTemplate.receiveAndConvert(CodeConstants.AMQP.MSG_QUEUE, new ParameterizedTypeReference<Msg>() {});

        log.info("message >> {}", message);
    }

```

<br/>

message 를 Dto 로 매핑하기 위해, rabbitmqTemplate 에 messageConverter를 추가하였다.

<br/>

