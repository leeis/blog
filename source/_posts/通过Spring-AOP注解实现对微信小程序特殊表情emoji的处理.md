---
title: 通过Spring AOP注解实现对微信小程序特殊表情emoji的处理
date: 2018-08-15 14:24:12
tags: Spring Boot
categories: Spring Boot
comments: true
toc: true
---

### Emoji
#### 1. Emoji是什么
- [Emoji](https://en.wikipedia.org/wiki/Emoji)就是表情符号，来自日语词汇“絵文字”（假名为“えもじ”，读音即 emoji）,  e (絵, "picture") + moji (文字, "character")。
- 最早由栗田穰崇（Shigetaka Kurita）创作，并在日本网络及手机用户中流行。 自苹果公司发布的 iOS 5 输入法中加入了 emoji 后，这种表情符号开始席卷全球。
- 目前 emoji 已被大多数现代计算机系统所兼容的 Unicode 编码采纳，普遍应用于各种手机短信和社交网络中。
- 一个emoji表情占4字节(32位)
- emoji不是图！不是图！不是图！emoji是字符！是字符！是字符！
- emoji可以任意粘贴复制，就像复制ABC123一样

#### 2. 在Mysql中保存Emoji
很多人在使用MYSQL时都是使用的utf8字符集，utf8只支持最多3个字节，而emoji需要4个字节，所以无法直接保存，直接保存会报错:
`SQLException: Incorrect string value: '\xF0\x9F\x98\x84' for column 'name' at row 1`

**解决方案1：直接过滤**
很多人面对错误的时候第一时间就是想着直接放弃emoji，不存储！
```java
public static String filter(String str){
    if (str == null) {  
       return null;  
   }  
   str = str.replaceAll("[^\\u0000-\\uFFFF]", "");    
  return str;  
}
```

**解决方案2：将MySQL编码从utf8转成utf8mb4**

其实utf8是mysql中的一个坑，mysql中真正的utf8应该是`utf8mb4`， utf8只支持最多3个字节，而真正的utf8每个字符最多是四个字节，MySQL一直没有修复这个bug，他们在2010年发布了一个叫做`utf8mb4`的字符集，绕过了这个问题，但是他们没有广而告之，以至于现在很多开发者仍然在使用utf8, 所以如果可以选择，`永远不要在MySQL中使用utf8，而应该用utf8mb4!`

所以当我们存储emoji表情的时候，需要将utf8转为`utf8mb4`， 具体操作如下([参考](https://mathiasbynens.be/notes/mysql-utf8mb4)):

- 1.Create a backup
- 2.Upgrade the MySQL server
- 3.Modify databases, tables, and columns
```
# For each database:
ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
# For each table:
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
# For each column:
ALTER TABLE table_name CHANGE column_name column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
# (Don’t blindly copy-paste this! The exact statement depends on the column type, maximum length, and other properties. The above line is just an example for a `VARCHAR` column.)
```

- 4.Check the maximum length of columns and index keys
- 5.Modify connection, client, and server character sets
> /etc/my.cnf

```
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

**验证**
`SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';`

- 6.Repair and optimize all tables

```
# For each table
REPAIR TABLE table_name;
OPTIMIZE TABLE table_name;
$ mysqlcheck -u root -p --auto-repair --optimize --all-databases
```
**解决方案3: 转成html**
emoji的编码有2种，分别是softbank和unicode, 我们通过网络上的[emoji库](https://raw.githubusercontent.com/googlei18n/emoji4unicode/master/data/emoji4unicode.xml), 将对应的Emoji转成[HTML Entity](https://developer.mozilla.org/en-US/docs/Glossary/Entity). Html Entity主要用来转义无法被正常表述的字符，以&开头，分号结尾，比如🍔 =`&#127828;`,可以通过 [amp-what](http://www.amp-what.com/unicode/search/) 进行转义查询

### 自定义Spring AOP注解
> 我们采用上述的方案3解决，通过加入**AOP注解**的方式，对方法参数自动进行转义，或者对返回结果自动进行转义。

我们将创建2个注解，`Emoji2Html`和`Emoji2Unicode`分别将Emoji转为Html，将Html转为Unicode.

#### 1.创建注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Emoji2Html {
}
```
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Emoji2Unicode {
}
```

#### 2.创建Aspect
这里我们利用Github上的一个[第三方Emoji处理工具](https://github.com/binarywang/java-emoji-converter), 我们用Aspect的`@Around`和`@AfterReturning`分别对参数和返回值进行处理，并且用`@annotation`绑定上我们定义的2个注解

`Emoji2Html`注解通过对方法的参数进行转换, 当参数类型为`String`或`Collection`时进行html转义

```java
@Around("@annotation(EmojiToHtml)")
    public Object toHtmlPoint(ProceedingJoinPoint jp) throws Throwable {
        Object[] args = jp.getArgs();
        //如果无参数，直接执行后返回
        if(args == null || args.length == 0) return jp.proceed();

        //有参数，将字符串参数进行转换后执行返回
        for (int i = 0; i < args.length; i++) {

            Object arg = args[i];

            if(arg instanceof String){
                args[i] = EmojiConverter.getInstance().toHtml((String)args[i]);
            } else if (arg instanceof Integer || arg instanceof Long || arg instanceof Float || arg instanceof Double || arg instanceof Boolean) {
                continue;
            } else if(arg instanceof Collection<?>){
                Iterator<?> iter = ((Collection) arg).iterator();
                while(iter.hasNext()){
                    this.checkObjectEmoji(iter.next(), false);
                }
            }else {
                this.checkObjectEmoji(arg, false);
            }
        }

        Object rvt = jp.proceed(args);
        return rvt;
    }
```

`toUnicodePoint`方法我们对返回值进行转换

```java
@AfterReturning(returning = "obj", pointcut = "@annotation(Emoji2Unicode)")
    public void toUnicodePoint(Object obj) {
        if (obj == null) return;
        if (obj instanceof String) {
            obj = EmojiConverter.getInstance().toUnicode((String) obj);
        } else if (obj instanceof Integer || obj instanceof Long || obj instanceof Float || obj instanceof Double || obj instanceof Boolean) {
            return;
        } else if (obj instanceof Collection<?>){

            Iterator<?> iter = ((Collection) obj).iterator();
            while(iter.hasNext()){
                this.checkObjectEmoji(iter.next(), true);
            }
        } else if (obj instanceof Map<?, ?>){
            return;
        } else{
            this.checkObjectEmoji(obj, true);
        }
    }
```

这个时候我们的注解就能用了，我们可以在我们想要使用的方法上添加注解即可

```java
public class LogService {

    private String name;

    @EmojiToHtml
    public void setName(String name){
        this.name = name;
        System.out.println(name);
    }

    @Emoji2Unicode
    public String getName(){
        return this.name;
    }
}
```

---

### 参考资料：
1. [Emojipedia](https://emojipedia.org/)
2. [Using Emojis in HTML, CSS, and JavaScript](https://www.kirupa.com/html5/emoji.htm)
2. [Implementing a Custom Spring AOP Annotation](https://www.baeldung.com/spring-aop-annotation)


