# Java反射获取Spring Bean中的注解

>  Spring框架中加载的bean是被Spring CGLib代理后的类实例，直接通过反射获取只能获取到代理类的属性，需要使用spring提供的工具类来获取真实的类实例，如果bean有实现接口那么还需要做实现类判断等操作。

1. 自定义Open注解类

```java
package cn.luojianbo.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;


@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Open {
    
    String value();
}

```

2. 以SpringBoot+Dubbo框架的为例，使用@DubboService注解的类会被Spring CGLib代理成为代理类（dubbo服务接口），这个代理类就是一个bean，同时我们在这个bean中使用自定义的注解@Open。通过以下代码可以获取到bean的真实类而非代理类，从而获取其中的自定义@Open注解

```java
package cn.luojianbo.servercommon.route;

import cn.luojianbo.servercommon.annotation.Open;
import org.apache.dubbo.config.annotation.DubboService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.core.annotation.AnnotatedElementUtils;
import org.springframework.core.annotation.AnnotationAttributes;
import org.springframework.core.annotation.AnnotationUtils;
import org.springframework.util.ClassUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.lang.annotation.Annotation;
import java.lang.reflect.Method;
import java.util.Arrays;
import java.util.Map;
import java.util.Objects;
import java.util.function.Function;
import java.util.stream.Collectors;

import static org.apache.dubbo.config.spring.util.DubboAnnotationUtils.resolveServiceInterfaceClass;

/**
 * @author luojianbo
 */
@RestController
public class FindAnnotationController {

    @Autowired
    private ApplicationContext applicationContext;


    @RequestMapping("/find")
    public void listRoutes() {
        this.findAnnotation();
    }

    private void findAnnotation() {
        Map<String, Object> dubboInterfaces = applicationContext.getBeansWithAnnotation(DubboService.class);
        for (Map.Entry<String, Object> entry : dubboInterfaces.entrySet()) {
            Object dubboInterface = entry.getValue();
            Class<?> aClass = dubboInterface.getClass();
            DubboService annotation = aClass.getAnnotation(DubboService.class);
            String group = annotation.group();
            String version = annotation.version();
            String tag = annotation.tag();
            int timeout = annotation.timeout();
            // 由于这里是spring cglib代理类注解已经丢失，重新获取类来判断open
            Class<?> userClass = ClassUtils.getUserClass(aClass);
            // 实现类可能实现了多个interface根据DubboService注解来获取正确的interface
            Annotation service = findServiceAnnotation(userClass, DubboService.class);
            AnnotationAttributes serviceAnnotationAttributes = AnnotationUtils.getAnnotationAttributes(service, false, false);
            Class<?> interfaceClass = resolveServiceInterfaceClass(serviceAnnotationAttributes, userClass);
            String interfaceName = interfaceClass.getName();
            Method[] interfaceMethods = interfaceClass.getDeclaredMethods();
            // 实际dubbo method以interface为准
            if (interfaceMethods.length == 0) {
                return;
            }
            Map<String, Method> subMethodMap = Arrays.stream(userClass.getDeclaredMethods()).collect(Collectors.toMap(this::buildMethodSign, Function.identity(), (o,n) -> n));
            for (Method method : interfaceMethods) {
                Method subMethod = subMethodMap.get(buildMethodSign(method));
                if (Objects.isNull(subMethod)) {
                    continue;
                }
                Open open = subMethod.getAnnotation(Open.class);
                // TODO: do something...
            }
        }
    }

    private Annotation findServiceAnnotation(Class<?> beanClass, Class<? extends Annotation> annotationType) {
        return AnnotatedElementUtils.findMergedAnnotation(beanClass, annotationType);
    }

    private String buildMethodSign(Method method) {
        String methodName = method.getName();
        Class<?>[] parameterTypes = method.getParameterTypes();
        if (parameterTypes.length == 0) {
            return methodName;
        }
        StringBuilder sb = new StringBuilder();
        sb.append(methodName);
        Arrays.stream(parameterTypes).map(Class::getName).forEach(sb::append);
        return sb.toString();
    }
}

```

