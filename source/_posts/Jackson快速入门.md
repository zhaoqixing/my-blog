---
title: jackson 快速入门
---
Java生态圈中有很多处理JSON和XML格式化的类库，Jackson是其中比较著名的一个。虽然JDK自带了XML处理类库，但是相对来说比较低级，使用本文介绍的Jackson等高级类库处理起来会方便很多。

###引入类库

    ext {
        jacksonVersion = '2.9.5'
    }
    
    dependencies {
        compile group: 'com.fasterxml.jackson.core', name: 'jackson-core', version: jacksonVersion
        compile group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: jacksonVersion
        compile group: 'com.fasterxml.jackson.core', name: 'jackson-annotations', version: jacksonVersion
        // 引入XML功能
        compile group: 'com.fasterxml.jackson.dataformat', name: 'jackson-dataformat-xml', version: jacksonVersion
        // 比JDK自带XML实现更高效的类库
        compile group: 'com.fasterxml.woodstox', name: 'woodstox-core', version: '5.1.0'
        // Java 8 新功能
        compile group: 'com.fasterxml.jackson.datatype', name: 'jackson-datatype-jsr310', version: jacksonVersion
        compile group: 'com.fasterxml.jackson.module', name: 'jackson-module-parameter-names', version: jacksonVersion
        compile group: 'com.fasterxml.jackson.datatype', name: 'jackson-datatype-jdk8', version: jacksonVersion
    
        compileOnly group: 'org.projectlombok', name: 'lombok', version: '1.16.22'
    }
    
###属性命名
@JsonProperty注解指定一个属性用于JSON映射，默认情况下映射的JSON属性与注解的属性名称相同，不过可以使用该注解的value值修改JSON属性名，该注解还有一个index属性指定生成JSON属性的顺序，如果有必要的话。

###属性包含
还有一些注解可以管理在映射JSON的时候包含或排除某些属性，下面介绍一下常用的几个。

@JsonIgnore注解用于排除某个属性，这样该属性就不会被Jackson序列化和反序列化。

@JsonIgnoreProperties注解是类注解。在序列化为JSON的时候，@JsonIgnoreProperties({"prop1", "prop2"})会忽略pro1和pro2两个属性。在从JSON反序列化为Java类的时候，@JsonIgnoreProperties(ignoreUnknown=true)会忽略所有没有Getter和Setter的属性。该注解在Java类和JSON不完全匹配的时候很有用。

@JsonIgnoreType也是类注解，会排除所有指定类型的属性。

###序列化相关
@JsonPropertyOrder和@JsonProperty的index属性类似，指定属性序列化时的顺序。

@JsonRootName注解用于指定JSON根属性的名称。
###处理JSON
###简单映射
我们用Lombok设置一个简单的Java类。

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class Friend {
        private String nickname;
        private int age;
    }
    
然后就可以处理JSON数据了。首先需要一个ObjectMapper对象，序列化和反序列化都需要它。

        ObjectMapper mapper = new ObjectMapper();
        Friend friend = new Friend("yitian", 25);

        // 写为字符串
        String text = mapper.writeValueAsString(friend);
        // 写为文件
        mapper.writeValue(new File("friend.json"), friend);
        // 写为字节流
        byte[] bytes = mapper.writeValueAsBytes(friend);
        System.out.println(text);
        // 从字符串中读取
        Friend newFriend = mapper.readValue(text, Friend.class);
        // 从字节流中读取
        newFriend = mapper.readValue(bytes, Friend.class);
        // 从文件中读取
        newFriend = mapper.readValue(new File("friend.json"), Friend.class);
        System.out.println(newFriend);
        
程序结果如下。可以看到生成的JSON属性和Java类中定义的一致。

    {"nickname":"yitian","age":25}
    Friend(nickname=yitian, age=25)
    
###集合的映射

除了使用Java类进行映射之外，我们还可以直接使用Map和List等Java集合组织JSON数据，在需要的时候可以使用readTree方法直接读取JSON中的某个属性值。需要注意的是从JSON转换为Map对象的时候，由于Java的类型擦除，所以类型需要我们手动用new TypeReference<T>给出。

        ObjectMapper mapper = new ObjectMapper();

        Map<String, Object> map = new HashMap<>();
        map.put("age", 25);
        map.put("name", "yitian");
        map.put("interests", new String[]{"pc games", "music"});

        String text = mapper.writeValueAsString(map);
        System.out.println(text);

        Map<String, Object> map2 = mapper.readValue(text, new TypeReference<Map<String, Object>>() {
        });
        System.out.println(map2);

        JsonNode root = mapper.readTree(text);
        String name = root.get("name").asText();
        int age = root.get("age").asInt();

        System.out.println("name:" + name + " age:" + age);
        
程序结果如下。

    {"name":"yitian","interests":["pc games","music"],"age":25}
    {name=yitian, interests=[pc games, music], age=25}
    name:yitian age:25
    
###Jackson配置
    
Jackson预定义了一些配置，我们通过启用和禁用某些属性可以修改Jackson运行的某些行为。详细文档参考JacksonFeatures。下面我简单翻译一下Jackson README上列出的一些属性。

    // 美化输出
    mapper.enable(SerializationFeature.INDENT_OUTPUT);
    // 允许序列化空的POJO类
    // （否则会抛出异常）
    mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
    // 把java.util.Date, Calendar输出为数字（时间戳）
    mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    
    // 在遇到未知属性的时候不抛出异常
    mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
    // 强制JSON 空字符串("")转换为null对象值:
    mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
    
    // 在JSON中允许C/C++ 样式的注释(非标准，默认禁用)
    mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
    // 允许没有引号的字段名（非标准）
    mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
    // 允许单引号（非标准）
    mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
    // 强制转义非ASCII字符
    mapper.configure(JsonGenerator.Feature.ESCAPE_NON_ASCII, true);
    // 将内容包裹为一个JSON属性，属性名由@JsonRootName注解指定
    mapper.configure(SerializationFeature.WRAP_ROOT_VALUE, true);
    
这里有三个方法，configure方法接受配置名和要设置的值，Jackson 2.5版本新加的enable和disable方法则直接启用和禁用相应属性，推荐使用后面两个方法。

###用注解管理映射

前面介绍了一些Jackson注解，下面来应用一下这些注解。首先来看看使用了注解的Java类。

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @JsonRootName("FriendDetail")
    @JsonIgnoreProperties({"uselessProp1", "uselessProp3"})
    public class FriendDetail {
        @JsonProperty("NickName")
        private String name;
        @JsonProperty("Age")
        private int age;
        private String uselessProp1;
        @JsonIgnore
        private int uselessProp2;
        private String uselessProp3;
    }
    
然后看看代码。需要注意的是，由于设置了排除的属性，所以生成的JSON和Java类并不是完全对应关系，所以禁用DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES是必要的。

        ObjectMapper mapper = new ObjectMapper();
        //mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
        mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
        FriendDetail fd = new FriendDetail("yitian", 25, "", 0, "");
        String text = mapper.writeValueAsString(fd);
        System.out.println(text);

        FriendDetail fd2 = mapper.readValue(text, FriendDetail.class);
        System.out.println(fd2);
        
运行结果如下。可以看到生成JSON的时候忽略了我们制定的值，而且在转换为Java类的时候对应的属性为空。

    {"NickName":"yitian","Age":25}
    FriendDetail(name=yitian, age=25, uselessProp1=null, uselessProp2=0, uselessProp3=null)
    
    
    
##处理XML

Jackson是一个处理JSON的类库，不过它也通过jackson-dataformat-xml包提供了处理XML的功能。Jackson建议我们在处理XML的时候使用woodstox-core包，它是一个XML的实现，比JDK自带XML实现更加高效，也更加安全。

这里有个注意事项，如果你正在使用Java 9以上的JDK，可能会出现java.lang.NoClassDefFoundError: javax/xml/bind/JAXBException异常，这是因为Java 9实现了JDK的模块化，将原本和JDK打包在一起的JAXB实现分隔出来。所以这时候需要我们手动添加JAXB的实现。在Gradle中添加下面的代码即可。

    compile group: 'javax.xml.bind', name: 'jaxb-api', version: '2.3.0'
    
###注解
    
Jackson XML除了使用Jackson JSON和JDK JAXB的一些注解之外，自己也定义了一些注解。下面简单介绍一下几个常用注解。

@JacksonXmlProperty注解有三个属性，namespace和localname属性用于指定XML命名空间的名称，isAttribute指定该属性作为XML的属性（）还是作为子标签（）.

@JacksonXmlRootElement注解有两个属性，namespace和localname属性用于指定XML根元素命名空间的名称。

@JacksonXmlText注解将属性直接作为未被标签包裹的普通文本表现。

@JacksonXmlCData将属性包裹在CDATA标签中。

###XML映射

新建如下一个Java类。
    
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @JsonRootName("Person")
    public class Person {
        @JsonProperty("Name")
        private String name;
        @JsonProperty("NickName")
        //@JacksonXmlText
        private String nickname;
        @JsonProperty("Age")
        private int age;
        @JsonProperty("IdentityCode")
        @JacksonXmlCData
        private String identityCode;
        @JsonProperty("Birthday")
        //@JacksonXmlProperty(isAttribute = true)
        @JsonFormat(pattern = "yyyy/MM/DD")
        private LocalDate birthday;
    
    }

下面是代码示例，基本上和JSON的API非常相似，XmlMapper实际上就是ObjectMapper的子类。

        Person p1 = new Person("yitian", "易天", 25, "10000", LocalDate.of(1994, 1, 1));
        XmlMapper mapper = new XmlMapper();
        mapper.findAndRegisterModules();
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        mapper.enable(SerializationFeature.INDENT_OUTPUT);
        String text = mapper.writeValueAsString(p1);
        System.out.println(text);

        Person p2 = mapper.readValue(text, Person.class);
        System.out.println(p2);
        
运行结果如下。

    <Person>
      <Name>yitian</Name>
      <NickName>易天</NickName>
      <Age>25</Age>
      <IdentityCode><![CDATA[10000]]></IdentityCode>
      <Birthday>1994/01/01</Birthday>
    </Person>
    
    Person(name=yitian, nickname=易天, age=25, identityCode=10000, birthday=1994-01-01)
    
如果取消那两行注释，那么运行结果如下。可以看到Jackson XML注解对生成的XML的控制效果。

    <Person birthday="1994/01/01">
      <Name>yitian</Name>易天
      <Age>25</Age>
      <IdentityCode><![CDATA[10000]]></IdentityCode>
    </Person>
    
    Person(name=yitian, nickname=null, age=25, identityCode=10000, birthday=1994-01-01)