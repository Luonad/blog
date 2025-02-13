# 20240604

> 记一次Spring Cloud Gateway的Filter重复执行两次的问题

- 背景：公司开放平台使用Spring Cloud Gateway搭建了的开放网关，统一所有开放资源接口路由。今天突然有客户反馈接口请求报错，错误信息是nonce参数重复导致的验签不通过，但客户确认使用的uuid最为nonce参数所以几乎不可能重复。
- 排查过程：
    - 根据异常定位代码位置，得出是nonce参数重复导致的验签不通过。nonce的作用类似于requestId避免重复的请求，代码中nonce使用redis缓存一小时
    - 使用客户提供的参数在生产稳定复现问题
    - 在测试环境debug发现验签的GatewayFilter（**MroGatewayFilter**）重复执行了两次，导致一次请求的nonce重复校验两次自然不通过
    - 经过半天的排查和debug最终发现问题，是由于代码中有个修改Response的GlobalFilter（**GatewayModifyResponseGatewayFilter**）存在Bug，在重写ResponseBody时又去调用了chain.filter(exchange)。这样就会导致重复执行Filter
- 代码如下

GatewayFilter（**MroGatewayFilter**）

```java
package cn.yzw.uop.gateway.filter.extend;

import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;


/**
 * @author luojianbo
 * @date 2023/11/8
 */
@Slf4j
@Component
public class MroGatewayFilter implements GatewayFilter, Ordered {
    
    @Override
    public int getOrder() {
        return 0;
    }

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 重复执行两次！！！
        // todo something......
        return chain.filter(exchange);
    }
}
```

GlobalFilter（**GatewayModifyResponseGatewayFilter**）存在Bug

```java
public class GatewayModifyResponseGatewayFilter implements GlobalFilter, Ordered {

    @Override
    @SuppressWarnings("unchecked")
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        WatchUtils.point(exchange, "GatewayModifyResponseGatewayFilter start");
        ServerHttpResponseDecorator responseDecorator = new ServerHttpResponseDecorator(exchange.getResponse()) {

            @Override
            public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {
                if (exchange.getAttribute(SopConstants.RESTFUL_REQUEST) != null) {
                    WatchUtils.point(exchange, "GatewayModifyResponseGatewayFilter end");
                    return chain.filter(exchange); // 问题就出在这里，这里只是一个Response装饰器不能去调用下一个过滤器，正确的做法实在filter方法去调用下一个过滤器chain.filter(exchange)
                }
                byte[] bits = JSONObject.toJSONString("新的body内容").getBytes(StandardCharsets.UTF_8);
                DataBuffer buffer = exchange.getResponse().bufferFactory().wrap(bits);
                exchange.getResponse().setStatusCode(HttpStatus.OK);
                exchange.getResponse().getHeaders().setContentType(MediaType.APPLICATION_JSON);
                return exchange.getResponse().writeWith(Mono.just(buffer));
            }

            @Override
            public Mono<Void> writeAndFlushWith(Publisher<? extends Publisher<? extends DataBuffer>> body) {
                return writeWith(Flux.from(body)
                        .flatMapSequential(p -> p));
            }
        };
        return chain.filter(exchange.mutate().response(responseDecorator).build());
    }
  
    @Override
    public int getOrder() {
        return NettyWriteResponseFilter.WRITE_RESPONSE_FILTER_ORDER - 1;
    }
} 
```

- 修复后的GatewayModifyResponseGatewayFilter

```java
public class GatewayModifyResponseGatewayFilter implements GlobalFilter, Ordered {

    @Override
    @SuppressWarnings("unchecked")
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        WatchUtils.point(exchange, "GatewayModifyResponseGatewayFilter start");
        ServerHttpResponseDecorator responseDecorator = new ServerHttpResponseDecorator(exchange.getResponse()) {

            @Override
            public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {
                byte[] bits = JSONObject.toJSONString("新的body内容").getBytes(StandardCharsets.UTF_8);
                DataBuffer buffer = exchange.getResponse().bufferFactory().wrap(bits);
                exchange.getResponse().setStatusCode(HttpStatus.OK);
                exchange.getResponse().getHeaders().setContentType(MediaType.APPLICATION_JSON);
                return exchange.getResponse().writeWith(Mono.just(buffer));
            }

            @Override
            public Mono<Void> writeAndFlushWith(Publisher<? extends Publisher<? extends DataBuffer>> body) {
                return writeWith(Flux.from(body)
                        .flatMapSequential(p -> p));
            }
        };
      if (exchange.getAttribute(SopConstants.RESTFUL_REQUEST) != null) {
                    WatchUtils.point(exchange, "GatewayModifyResponseGatewayFilter end");
                    return chain.filter(exchange); // 在这里去调用下一个过滤器才是正确的
                }
        return chain.filter(exchange.mutate().response(responseDecorator).build());
    }
  
    @Override
    public int getOrder() {
        return NettyWriteResponseFilter.WRITE_RESPONSE_FILTER_ORDER - 1;
    }
}  
```

- 总结

1. 此次问题的出现是由于修改代码后未进行回归测试导致的线上bug，好在影响范围较小。后续修改历史代码还需要思考再三考虑影响范围并做好回归测试。
2. 关于过滤器重复执行的问题应当首先怀疑调用链路执行问题，检查代码是否在过滤器中的重复调用下一个过滤器。
3. GatewayFilter和GlobalFilter是SpringCloudGateway的两种过滤器，局部过滤器和全局过滤器。