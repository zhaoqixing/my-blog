---
title: commons-lang3-3.2.jar中的常用工具类的使用
---
#### 1、StringUtils工具类

可以判断是否是空串，是否为null，默认值设置等操作:

    /**
         * StringUtils
         */
        public static void test1() {
            System.out.println(StringUtils.isBlank("   "));// true----可以验证null, ""," "等
            System.out.println(StringUtils.isBlank("null"));// false
            System.out.println(StringUtils.isAllLowerCase("null"));// t
            System.out.println(StringUtils.isAllUpperCase("XXXXXX"));// t
            System.out.println(StringUtils.isEmpty(" "));// f---为null或者""返回true
            System.out.println(StringUtils.defaultIfEmpty(null, "default"));// 第二个参数是第一个为null或者""的时候的取值
            System.out.println(StringUtils.defaultIfBlank("    ", "default"));//// 第二个参数是第一个为null或者""或者"   "的时候的取值
        }
        
isBlank()  可以验证空格、null、""，如果是好几个空格也返回true

isEmpty验证不了空格，只有值为null和""返回true 

两者都验证不了"null"字符串，所以如果验证"null"还需要自己用equals进行验证。

    结果:
    
    true
    false
    true
    true
    false
    default
    default

简单的贴出几个源码便于记录:

    public static boolean isBlank(final CharSequence cs) {
            int strLen;
            if (cs == null || (strLen = cs.length()) == 0) {
                return true;
            }
            for (int i = 0; i < strLen; i++) {
                if (Character.isWhitespace(cs.charAt(i)) == false) {
                    return false;
                }
            }
            return true;
        }
    
        public static boolean isEmpty(final CharSequence cs) {
            return cs == null || cs.length() == 0;
        }
        
        
           public static String defaultIfEmpty(String str, String defaultStr) {
                return StringUtils.isEmpty(str) ? defaultStr : str;
            }
            
CharSequence是一个接口，String,StringBuffer,StringBuilder等都实现了此接口

    public abstract interface CharSequence {
        public abstract int length();
    
        public abstract char charAt(int paramInt);
    
        public abstract CharSequence subSequence(int paramInt1, int paramInt2);
    
        public abstract String toString();
    }
    
补充:StringUtils页可以将集合转为String，并且以指定符号链接里面的数据

    List list = new ArrayList(2);
            list.add("张三");
            list.add("李四");
            list.add("王五");
            String list2str = StringUtils.join(list, ",");
            System.out.println(list2str);
            
    结果:
    
    　　张三,李四,王五

 

补充:有时候我们希望给拼接后的字符串都加上单引号，这个在拼接SQL  in条件的时候非常有用，例如:

        //需求:将逗号里面的内容都加上单引号
        String string = "111,222,333";
        string = "'"+string+"'";//字符串前后加'
        string = StringUtils.join(string.split(","),"','");//先按逗号分隔为数组，然后用','连接数组
        System.out.println(string);
        
    结果:
    
    '111','222','333'

补充:String.format(format,Object)也可以对字符串进行格式化，例如在数字前面补齐数字

            int num = 50;
            String format = String.format("%0" + 5 + "d", num);
            System.out.println(format);
            
    结果:
    
    00050

补充:StringUtils也可以截取字符串,判断是否大小写等操作

    String string = "123_45_43_ss";
            System.out.println(StringUtils.isAllLowerCase(string));// 判断全部小写
            System.out.println(StringUtils.isAllUpperCase(string));// 判断全部大写
            System.out.println(StringUtils.substringAfter(string, "123"));// 截取123之后的
            System.out.println(StringUtils.substringBefore(string, "45"));// 截取45之前的
            System.out.println(StringUtils.substringBefore(string, "_"));// 截取第一个_之前的
            System.out.println(StringUtils.substringBeforeLast(string, "_"));// 截取最后一个_之前的
            System.out.println(StringUtils.substringAfter(string, "_"));// 截取第一个_之后的
            System.out.println(StringUtils.substringAfterLast(string, "_"));// 截取最后一个_之后的
            System.out.println(StringUtils.substringBetween("1234565432123456", "2", "6"));// 截取两个之间的(都找的是第一个)
            
#### 2.StringEscapeUtils----------转义字符串的工具类

    /**
         * StringEscapeUtils
         */
        public static  void test2(){
            //1.防止sql注入------原理是将'替换为''
            System.out.println(org.apache.commons.lang.StringEscapeUtils.escapeSql("sss"));
            //2.转义/反转义html
            System.out.println( org.apache.commons.lang.StringEscapeUtils.escapeHtml("<a>dddd</a>"));   //&lt;a&gt;dddd&lt;/a&gt;
            System.out.println(org.apache.commons.lang.StringEscapeUtils.unescapeHtml("&lt;a&gt;dddd&lt;/a&gt;"));  //<a>dddd</a>
            //3.转义/反转义JS
            System.out.println(org.apache.commons.lang.StringEscapeUtils.escapeJavaScript("<script>alert('1111')</script>"));   
            //4.把字符串转为unicode编码
            System.out.println(org.apache.commons.lang.StringEscapeUtils.escapeJava("中国"));   
            System.out.println(org.apache.commons.lang.StringEscapeUtils.unescapeJava("\u4E2D\u56FD"));  
            //5.转义JSON
            System.out.println(org.apache.commons.lang3.StringEscapeUtils.escapeJson("{name:'qlq'}"));   
        }
        
#### 3.NumberUtils--------字符串转数据或者判断字符串是否是数字常用工具类

    /**
         * NumberUtils
         */
        public static  void test3(){
            System.out.println(NumberUtils.isNumber("231232.8"));//true---判断是否是数字
            System.out.println(NumberUtils.isDigits("2312332.5"));//false，判断是否是整数
            System.out.println(NumberUtils.toDouble(null));//如果传的值不正确返回一个默认值，字符串转double，传的不正确会返回默认值
            System.out.println(NumberUtils.createBigDecimal("333333"));//字符串转bigdecimal
        }
        
#### 4.BooleanUtils------------判断Boolean类型工具类

    /**
         * BooleanUtils
         */
        public static  void test4(){
            System.out.println(BooleanUtils.isFalse(true));//false
            System.out.println(BooleanUtils.toBoolean("yes"));//true
            System.out.println(BooleanUtils.toBooleanObject(0));//false
            System.out.println(BooleanUtils.toStringYesNo(false));//no
            System.out.println(BooleanUtils.toBooleanObject("ok", "ok", "error", "null"));//true-----第一个参数是需要验证的字符串，第二个是返回true的值，第三个是返回false的值，第四个是返回null的值
        }
        
#### 5.SystemUtils----获取系统信息(原理都是调用System.getProperty())

    /**
         * SystemUtils
         */
        public static  void test5(){
            System.out.println(SystemUtils.getJavaHome());
            System.out.println(SystemUtils.getJavaIoTmpDir());
            System.out.println(SystemUtils.getUserDir());
            System.out.println(SystemUtils.getUserHome());
            System.out.println(SystemUtils.JAVA_VERSION);
            System.out.println(SystemUtils.OS_NAME);
            System.out.println(SystemUtils.USER_TIMEZONE);
        }
        
#### 6.DateUtils和DateFormatUtils可以实现字符串转date与date转字符串,date比较先后问题

DateUtils也可以判断是否是同一天等操作。

    package zd.dms.test;
    
    import java.text.ParseException;
    import java.util.Date;
    
    import org.apache.commons.lang3.time.DateFormatUtils;
    import org.apache.commons.lang3.time.DateUtils;
    
    public class PlainTest {
        public static void main(String[] args) {
            // DateFormatUtils----date转字符串
            Date date = new Date();
            System.out.println(DateFormatUtils.format(date, "yyyy-MM-dd hh:mm:ss"));// 小写的是12小时制
            System.out.println(DateFormatUtils.format(date, "yyyy-MM-dd HH:mm:ss"));// 大写的HH是24小时制
    
            // DateUtils ---加减指定的天数(也可以加减秒、小时等操作)
            Date addDays = DateUtils.addDays(date, 2);
            System.out.println(DateFormatUtils.format(addDays, "yyyy-MM-dd HH:mm:ss"));
            Date addDays2 = DateUtils.addDays(date, -2);
            System.out.println(DateFormatUtils.format(addDays2, "yyyy-MM-dd HH:mm:ss"));
    
            // 原生日期判断日期先后顺序
            System.out.println(addDays2.after(addDays));
            System.out.println(addDays2.before(addDays));
    
            // DateUtils---字符串转date
            String strDate = "2018-11-01 19:23:44";
            try {
                Date parseDateStrictly = DateUtils.parseDateStrictly(strDate, "yyyy-MM-dd HH:mm:ss");
                Date parseDate = DateUtils.parseDate(strDate, "yyyy-MM-dd HH:mm:ss");
                System.out.println(parseDateStrictly);
                System.out.println(parseDate);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }
    }
    
#### 7.StopWatch提供秒表的计时,暂停等功能

    package cn.xm.exam.test;
    
    import org.apache.commons.lang.time.StopWatch;
    
    public class test implements AInterface, BInterface {
        public static void main(String[] args) {
            StopWatch stopWatch = new StopWatch();
            stopWatch.start();
    
            try {
                Thread.sleep(5 * 1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
    
            stopWatch.stop();
            System.out.println(stopWatch.getStartTime());// 获取开始时间
            System.out.println(stopWatch.getTime());// 获取总的执行时间--单位是毫秒
        }
    }

#### 8.以Range结尾的类主要提供一些范围的操作,包括判断某些字符,数字等是否在这个范围以内

            IntRange intRange = new IntRange(1, 5);
            System.out.println(intRange.getMaximumInteger());
            System.out.println(intRange.getMinimumInteger());
            System.out.println(intRange.containsInteger(6));
            System.out.println(intRange.containsDouble(3));
            
    结果:
    
    5
    1
    false
    true

#### 9.ArrayUtils操作数组，功能强大，可以合并，判断是否包含等操作

    package cn.xm.exam.test;
    
    import org.apache.commons.lang.ArrayUtils;
    
    public class test implements AInterface, BInterface {
        public static void main(String[] args) {
            int array[] = { 1, 5, 5, 7 };
            System.out.println(array);
    
            // 增加元素
            array = ArrayUtils.add(array, 9);
            System.out.println(ArrayUtils.toString(array));
    
            // 删除元素
            array = ArrayUtils.remove(array, 3);
            System.out.println(ArrayUtils.toString(array));
    
            // 反转数组
            ArrayUtils.reverse(array);
            System.out.println(ArrayUtils.toString(array));
    
            // 查询数组索引
            System.out.println(ArrayUtils.indexOf(array, 5));
    
            // 判断数组中是否包含指定值
            System.out.println(ArrayUtils.contains(array, 5));
    
            // 合并数组
            array = ArrayUtils.addAll(array, new int[] { 1, 5, 6 });
            System.out.println(ArrayUtils.toString(array));
        }
    }

#### 8.  反射工具类的使用

一个普通的java:

    package cn.xm.exam.test.p1;
    
    public class Person {
        private String name;
    
        public static void staticMet(String t) {
            System.out.println(t);
        }
    
        public Person(String name) {
            this.name = name;
        }
    
        public String call(String string) {
            System.out.println(name);
            System.out.println(string);
            return string;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        @Override
        public String toString() {
            return "test [name=" + name + "]";
        }
    }
    
反射工具类操作:

    package cn.xm.exam.test;
    
    import java.lang.reflect.Constructor;
    import java.lang.reflect.Field;
    import java.lang.reflect.InvocationTargetException;
    import java.lang.reflect.Method;
    
    import org.apache.commons.lang.reflect.ConstructorUtils;
    import org.apache.commons.lang.reflect.FieldUtils;
    import org.apache.commons.lang.reflect.MethodUtils;
    
    import cn.xm.exam.test.p1.Person;
    
    public class test {
    
        public static void main(String[] args) throws InstantiationException, IllegalAccessException,
                IllegalArgumentException, InvocationTargetException, NoSuchMethodException {
            // ConstructorUtils工具类的使用
            Constructor accessibleConstructor = ConstructorUtils.getAccessibleConstructor(Person.class, String.class);
            Person newInstance = (Person) accessibleConstructor.newInstance("test");
            System.out.println(newInstance.getClass());
            System.out.println(newInstance);
    
            // MethodUtils的使用
            Method accessibleMethod = MethodUtils.getAccessibleMethod(Person.class, "call", String.class);
            Object invoke = accessibleMethod.invoke(newInstance, "参数");
            System.out.println(invoke);
            // 调用静态方法
            MethodUtils.invokeStaticMethod(Person.class, "staticMet", "静态方法");
    
            // FieldUtils 暴力获取私有变量(第三个参数表示是否强制获取)---反射方法修改元素的值
            Field field = FieldUtils.getField(Person.class, "name", true);
            field.setAccessible(true);
            System.out.println(field.getType());
            field.set(newInstance, "修改后的值");
            System.out.println(newInstance.getName());
        }
    }
    
    
    
    结果:
    
    class cn.xm.exam.test.p1.Person
    test [name=test]
    test
    参数
    参数
    静态方法
    class java.lang.String
    修改后的值

#### 9.  EqualsBuilder 可以用于拼接多个条件进行equals比较

    EqualsBuilder equalsBuilder = new EqualsBuilder();
            Integer integer1 = new Integer(1);
            Integer integer2 = new Integer(1);
    
            String string1 = "111";
            String string2 = "111";
            equalsBuilder.append(integer1, integer2);
            equalsBuilder.append(string1, string2);
            System.out.println(equalsBuilder.isEquals());
            
结果:

true

#### ========下面是commons-collections包中的常用的工具类==

#### 1. CollectionUtils工具类用于操作集合,  isEmpty () 方法最有用   （commons-collections包中的类）

    package cn.xm.exam.test;
    
    import java.util.ArrayList;
    import java.util.Collection;
    import java.util.List;
    
    import org.apache.commons.collections.CollectionUtils;
    
    public class test {
        public static void main(String[] args) {
            List<String> list = new ArrayList<String>();
            list.add("str1");
            list.add("str2");
    
            List<String> list1 = new ArrayList<String>();
            list1.add("str1");
            list1.add("str21");
    
            // 判断是否有任何一个相同的元素
            System.out.println(CollectionUtils.containsAny(list, list1));
    
            // 求并集(自动去重)
            List<String> list3 = (List<String>) CollectionUtils.union(list, list1);
            System.out.println(list3);
    
            // 求交集(两个集合中都有的元素)
            Collection intersection = CollectionUtils.intersection(list, list1);
            System.out.println("intersection->" + intersection);
    
            // 求差集(并集去掉交集，也就是list中有list1中没有，list1中有list中没有)
            Collection intersection1 = CollectionUtils.disjunction(list, list1);
            System.out.println("intersection1->" + intersection1);
    
            // 获取一个同步的集合
            Collection synchronizedCollection = CollectionUtils.synchronizedCollection(list);
    
            // 验证集合是否为null或者集合的大小是否为0，同理有isNouEmpty方法
            List list4 = null;
            List list5 = new ArrayList<>();
            System.out.println(CollectionUtils.isEmpty(list4));
            System.out.println(CollectionUtils.isEmpty(list5));
        }
    }
    
#### 2.   MapUtils工具类

可以用于map判断null和size为0，也可以直接获取map中的值为指定类型，没有的返回null

    package cn.xm.exam.test;
    
    import java.util.HashMap;
    import java.util.Map;
    
    import org.apache.commons.collections.MapUtils;
    import org.apache.commons.lang.NumberUtils;
    
    import ognl.MapElementsAccessor;
    
    public class test {
        public static void main(String[] args) {
            Map map = null;
            Map map2 = new HashMap();
            Map map3 = new HashMap<>();
            map3.put("xxx", "xxx");
            // 检验为empty可以验证null和size为0的情况
            System.out.println(MapUtils.isEmpty(map));
            System.out.println(MapUtils.isEmpty(map2));
            System.out.println(MapUtils.isEmpty(map3));
    
            String string = MapUtils.getString(map3, "eee");
            String string2 = MapUtils.getString(map3, "xxx");
            Integer integer = MapUtils.getInteger(map3, "xxx");
            System.out.println("string->" + string);
            System.out.println("string2->" + string2);
            System.out.println("integer->" + integer);
            System.out.println(integer == null);
        }
    }
    
结果:

    true
    true
    false
    INFO: Exception: java.text.ParseException: Unparseable number: "xxx"
    string->null
    string2->xxx
    integer->null
    true
    
补充:MapUtils也可以获取值作为String，获取不到取默认值:

            //获取字符串,如果获取不到可以返回一个默认值
            String string3 = MapUtils.getString(map3, "eee","没有值");
            



