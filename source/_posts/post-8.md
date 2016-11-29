---
title: Spring mvc返回JSON数据的两种配置方式
date: 2016-11-29 13:42:32
categories: Spring
---

### 1.视图解析方式
依赖包：jackson-core、jackson-databind、jackson-annotation
Spring配置文件内容：
```
<bean id="contentNegotiationManager" class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
        <property name="defaultContentType" value="application/json;charset=utf-8" ></property>
    </bean>
    <bean id="viewResolver" class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
        <property name="contentNegotiationManager" ref="contentNegotiationManager"></property>
        <property name="defaultViews">
            <list>
                <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView">
                    <property name="prettyPrint" value="true"/>
                </bean>
            </list>
        </property>
    </bean>
```
Controller代码：
```
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * Created by carrey on 16-11-29.
 */
@Controller
public class TestController {
    @RequestMapping(value="/test.do")
    public void test(Model m){
        m.addAttribute("test","Hello World!");
    }
}
```

### 2.@ResponseBody返回方式
依赖包：jackson-core、jackson-databind、jackson-annotation
Spring配置文件内容：
```
<mvc:annotation-driven/>
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
        <property name="messageConverters">
            <list>
                <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                    <property name="defaultCharset" value="utf-8"></property>
                </bean>
            </list>
        </property>
    </bean>
```
Controller代码：
```
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.HashMap;
import java.util.Map;

/**
 * Created by carrey on 16-11-29.
 */
@Controller
public class TestController {
    @ResponseBody
    @RequestMapping(value="/test.do")
    public Map test(){
        return new HashMap(){{put("aa","Hello World!");}};
    }
}
```

