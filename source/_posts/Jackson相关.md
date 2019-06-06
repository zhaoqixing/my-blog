---
title: jackson 相关认识
---
### spring是如何将string 通过jackson变成java对象的
 
Spring Boot支持与三种JSON mapping库集成：Gson、Jackson和JSON-B。Jackson是首选和默认的。

Jackson是spring-boot-starter-json的一部分，spring-boot-starter-web中包含spring-boot-starter-json。也就是说，当项目中引入spring-boot-starter-web后会自动引入spring-boot-starter-json。


       <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		
ObjectMapper是jackson-databind包中的一个类，提供读写JSON的功能，可以方便的进行对象和JSON转换：

				import com.fasterxml.jackson.core.JsonProcessingException;
				import com.fasterxml.jackson.databind.ObjectMapper;

				import java.io.IOException;

				public final class JsonUtil {
					private static ObjectMapper mapper = new ObjectMapper();

					private JsonUtil() {
					}

					/**
					 * Serialize any Java value as a String.
					 */
					public static String generate(Object object) throws JsonProcessingException {
						return mapper.writeValueAsString(object);
					}

					/**
					 * Deserialize JSON content from given JSON content String.
					 */
					public static <T> T parse(String content, Class<T> valueType) throws IOException {
						return mapper.readValue(content, valueType);
					}
				}	
				
编写一简单POJO测试类：

    				import java.util.Date;
    
    				public class Hero {
    
    					public static void main(String[] args) throws Exception {
    						System.out.println(JsonUtil.generate(new Hero("Jason", new Date())));
    					}
    
    					private String name;
    
    					private Date birthday;
    
    					public Hero() {
    					}
    
    					public Hero(String name, Date birthday) {
    						this.name = name;
    						this.birthday = birthday;
    					}
    
    					public String getName() {
    						return name;
    					}
    
    					public Date getBirthday() {
    						return birthday;
    					}
    				} 
    				
运行后输出结果如下：
		
			   {"name":"Jason","birthday":1540909420353}	
			   
#### 关于使用Feign的feign-master包中通过feign.jackson.JacksonEncoder、feign.jackson.JacksonDecoder类基于jackson进行加密解密

源码说明

    1.JacksonEncoder加密类
    
		package feign.jackson;
		 
		import com.fasterxml.jackson.annotation.JsonInclude;
		import com.fasterxml.jackson.core.JsonProcessingException;
		import com.fasterxml.jackson.databind.JavaType;
		import com.fasterxml.jackson.databind.Module;
		import com.fasterxml.jackson.databind.ObjectMapper;
		import com.fasterxml.jackson.databind.SerializationFeature;
		 
		import java.lang.reflect.Type;
		import java.util.Collections;
		 
		import feign.RequestTemplate;
		import feign.codec.EncodeException;
		import feign.codec.Encoder;
		 
		public class JacksonEncoder implements Encoder {
		 
		  private final ObjectMapper mapper;
		 
		  public JacksonEncoder() {
			this(Collections.<Module>emptyList());
		  }
		 
		  public JacksonEncoder(Iterable<Module> modules) {
			this(new ObjectMapper()
					 .setSerializationInclusion(JsonInclude.Include.NON_NULL)
					 .configure(SerializationFeature.INDENT_OUTPUT, true)
					 .registerModules(modules));
		  }
		 
		  public JacksonEncoder(ObjectMapper mapper) {
			this.mapper = mapper;
		  }
		 
		  @Override
		  public void encode(Object object, Type bodyType, RequestTemplate template) {
			try {
			  JavaType javaType = mapper.getTypeFactory().constructType(bodyType);
			  template.body(mapper.writerFor(javaType).writeValueAsString(object));
			} catch (JsonProcessingException e) {
			  throw new EncodeException(e.getMessage(), e);
			}
		  }
		}
		
2.JacksonDecoder解密类

		package feign.jackson;
		 
		import com.fasterxml.jackson.databind.DeserializationFeature;
		import com.fasterxml.jackson.databind.Module;
		import com.fasterxml.jackson.databind.ObjectMapper;
		import com.fasterxml.jackson.databind.RuntimeJsonMappingException;
		 
		import java.io.BufferedReader;
		import java.io.IOException;
		import java.io.Reader;
		import java.lang.reflect.Type;
		import java.util.Collections;
		 
		import feign.Response;
		import feign.Util;
		import feign.codec.Decoder;
		 
		public class JacksonDecoder implements Decoder {
		 
		  private final ObjectMapper mapper;
		 
		  public JacksonDecoder() {
			this(Collections.<Module>emptyList());
		  }
		 
		  public JacksonDecoder(Iterable<Module> modules) {
			this(new ObjectMapper().configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
					 .registerModules(modules));
		  }
		 
		  public JacksonDecoder(ObjectMapper mapper) {
			this.mapper = mapper;
		  }
		 
		  @Override
		  public Object decode(Response response, Type type) throws IOException {
			if (response.status() == 404) return Util.emptyValueOf(type);
			if (response.body() == null) return null;
			Reader reader = response.body().asReader();
			if (!reader.markSupported()) {
			  reader = new BufferedReader(reader, 1);
			}
			try {
			  // Read the first byte to see if we have any data
			  reader.mark(1);
			  if (reader.read() == -1) {
				return null; // Eagerly returning null avoids "No content to map due to end-of-input"
			  }
			  reader.reset();
			  return mapper.readValue(reader, mapper.constructType(type));
			} catch (RuntimeJsonMappingException e) {
			  if (e.getCause() != null && e.getCause() instanceof IOException) {
				throw IOException.class.cast(e.getCause());
			  }
			  throw e;
			}
		  }
		} 
		
#### ObjectMapper 是线程安全的吗？

是的，官方注释中这样回应的：

    	Is ObjectMapper thread-safe? Short answer: yes Long answer: yes, as long as you always configure instance before use, and do not call configure 
    	methods during operation (or synchronize such calls appropriately). Usually it is better to construct separate mapper instance if configurations differ in any case.   
    	
#### 为什么spring可以正确解析 application/jason  application/form 等

@RequestBody

该注解常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等；

它是通过使用HandlerAdapter 配置的HttpMessageConverters来解析post data body，然后绑定到相应的bean上的。

因为配置有FormHttpMessageConverter，所以也可以用来处理 application/x-www-form-urlencoded的内容，处理完的结果放在一个MultiValueMap<String, String>里


@RestController中有@ResponseBody，可以帮我们把对象列化到resp.body中。@RequestBody可以帮我们把req.body的内容转化为对象。如果是开发Web应用，一般
这两个注解对应的就是Json序列化和反序列化的操作。这里实际上已经体现了Http序列化/反序列化这个过程，只不过和普通的对象序列化有些不一样，Http序列化/反序
列化的层次更高，属于一种Object2Object之间的转换。
Http序列化和反序列化的核心是HttpMessageConverter。用过老版本springmvc的可能有些印象，那时候需要在xml配置文件中注入MappingJackson2HttpMessageConverter
这个类型的bean，告诉springmvc我们需要进行Json格式的转换，它就是HttpMessageConverter的一种实现。


#### spring支持哪些json解析，如果做，才能将spring默认的json解析工具变成 fastjson

Spring Boot支持与三种JSON mapping库集成：Gson、Jackson和JSON-B。Jackson是首选和默认的。
如果使用fastjson，可以按照下列方式配置使用
	
引入fastjson依赖库

	<dependencies>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.46</version>
		</dependency>
	</dependencies>
	
	compile("com.alibaba:fastjson:${fastJsonVersion}")
	ext {
			fastJsonVersion    = '1.2.46'
		}
		
注： 这里要说下很重要的话，官方文档说的1.2.10以后，会有两个方法支持HttpMessageconvert，一个是FastJsonHttpMessageConverter，支持4.2以下的版本，
		一个是FastJsonHttpMessageConverter4支持4.2以上的版本，具体有什么区别暂时没有深入研究。这里也就是说：低版本的就不支持了，所以这里最低要求就是1.2.10+
		
在启动类中配置

配置方式一（通过继承的方式）

　　1、启动类继承WebMvcConfigurerAdapter
　　2、重写configureMessageConverters方法

	@SpringBootApplication
	@EnableDiscoveryClient
	@EnableScheduling
	public class MemberApplication extends WebMvcConfigurerAdapter {
		/**
		 * 配置FastJson为方式一
		 * @return*/
		@Override
		public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
			super.configureMessageConverters(converters);
			/*
			 * 1、需要先定义一个convert转换消息的对象 2、添加fastJson的配置信息，比如：是否要格式化返回json数据 3、在convert中添加配置信息
			 * 4、将convert添加到converters当中
			 * 
			 */
			// 1、需要先定义一个·convert转换消息的对象；
			FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
			// 2、添加fastjson的配置信息，比如 是否要格式化返回json数据
			FastJsonConfig fastJsonConfig = new FastJsonConfig();
			fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
			// 3、在convert中添加配置信息.
			fastConverter.setFastJsonConfig(fastJsonConfig);
			// 4、将convert添加到converters当中.
			converters.add(fastConverter);
		}

		public static void main(String[] args) {
			SpringApplication.run(MemberApplication.class, args);
		}

	}
	
注：开发中为了统一管理配置，可以放入配置类中，启动类只做启动的功能

	@Configuration
	public class HttpConverterConfig {

		@Bean
		public HttpMessageConverters fastJsonHttpMessageConverters() {
			// 1.定义一个converters转换消息的对象
			FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
			// 2.添加fastjson的配置信息，比如: 是否需要格式化返回的json数据
			FastJsonConfig fastJsonConfig = new FastJsonConfig();
			fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
			// 3.在converter中添加配置信息
			fastConverter.setFastJsonConfig(fastJsonConfig);
			// 4.将converter赋值给HttpMessageConverter
			HttpMessageConverter<?> converter = fastConverter;
			// 5.返回HttpMessageConverters对象
			return new HttpMessageConverters(converter);
		}
	}
	
配置方式二（通过@Bean注入的方式）
在App.java启动类中，注入Bean : HttpMessageConverters

	@SpringBootApplication
	@EnableDiscoveryClient
	@EnableScheduling
	public class MemberApplication {
		/**
		 * 配置FastJson方式二
		 * @return  HttpMessageConverters
		 */
		@Bean
		public HttpMessageConverters fastJsonHttpMessageConverters() {
			// 1.定义一个converters转换消息的对象
			FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
			// 2.添加fastjson的配置信息，比如: 是否需要格式化返回的json数据
			FastJsonConfig fastJsonConfig = new FastJsonConfig();
			fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
			// 3.在converter中添加配置信息
			fastConverter.setFastJsonConfig(fastJsonConfig);
			// 4.将converter赋值给HttpMessageConverter
			HttpMessageConverter<?> converter = fastConverter;
			// 5.返回HttpMessageConverters对象
			return new HttpMessageConverters(converter);
		}
		
		public static void main(String[] args) {
			SpringApplication.run(MemberApplication.class, args);
		}

	}
	
在pojo类中：

    	private int id;
        private String name;
        
        //com.alibaba.fastjson.annotation.JSONField
        @JSONField(format="yyyy-MM-dd HH:mm")
        private Date createTime;//创建时间.
        
        /*
         * serialize:是否需要序列化属性.
         */
        @JSONField(serialize=false)
        private String remarks;//备注信息.
    	
那么这时候在实体类中使用@JSONField(serialize=false)，是不是此字段就不返回了，如果是的话，那么就配置成功了，其中JSONField的包路径是：com.alibaba.fastjson.annotation.JSONField。