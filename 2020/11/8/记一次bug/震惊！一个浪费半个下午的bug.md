### 震惊！一个浪费半个下午的bug

**bug出现场景：**

当再利用maven使用mybatis的时候，一切配置都准备好，要启动项目的时候，突然报错：

```java
Exception in thread "main" org.apache.ibatis.exceptions.PersistenceException: 
### Error building SqlSession.
### The error may exist in com/kyg/mapper/xml/OrderMapper.xml
### Cause: org.apache.ibatis.builder.BuilderException: Error parsing SQL Mapper Configuration. Cause: java.io.IOException: Could not find resource com/kyg/mapper/xml/OrderMapper.xml
```

看到这个提示，根据不怕bug的原则，先阅读错误信息，大致知道是没有找到OrderMapper.xml.这个映射器，我们是在mybatis-config.xml配置的，因此，查看这个文件：

```java
<mappers>
        <mapper resource="com/kyg/mapper/xml/OrderMapper.xml"/>
        <mapper resource="com/kyg/mapper/xml/ProductMapper.xml"/>
    </mappers>
```

**解决过程：**

仔细看看，确定再确定，也没发现有啥错，抓狂了很久，想想这个是静态资源，以前使用springboot使用html,css这些静态资源的时候，也出现了访问不到的情况，以前那是因为springboot框架对可能会对某些静态资源进行过滤，需要手动地配置哪些不过滤。那么这里只有mybatis和maven,mybatis应该不会对自己资源进行过滤，那么就是maven,于是查询了下maven对静态资源的处理，发现：

**maven构建项目时候，会对某些静态资源如css,html,xml进行过滤，因此我们的OrderMapper.xml可能被过滤掉了**，那么如何忽略过滤，自然而然通过pom文件配置，利用<resources>节点进行配置。配置如下：

```java
<bulid>
<resources>
      <resource>
        <directory>src/main/java</directory>
        <includes>
          <include>**/*.properties</include>
          <include>**/*.xml</include>
        </includes>
        <filtering>false</filtering>
      </resource>
      <resource>
        <directory>src/main/resources</directory>
        <includes>
          <include>**/*.properties</include>
          <include>**/*.xml</include>
        </includes>
        <filtering>false</filtering>
      </resource>
    </resources>
    </bulid>
```

配置过后顺利解决！！！

所以遇到静态资源访问不到，可以往是否被某些框架或工具过滤掉的问题。