---
title: YAML教程
date: 2021-09-04 10:59:43
tags: [Spring Boot]
categories: [Spring Boot]
---

Spring Boot 提供了大量的自动配置，极大地简化了spring 应用的开发过程，当用户创建了一个 Spring Boot 项目后，即使不进行任何配置，该项目也能顺利的运行起来。当然，用户也可以根据自身的需要使用配置文件修改 Spring Boot 的默认设置。

SpringBoot 默认使用以下 2 种全局的配置文件，其文件名是固定的。
* `application.properties`
* `application.yml`

其中，`application.yml`是一种使用 YAML 语言编写的文件，它与`application.properties`一样，可以在 Spring Boot 启动时被自动读取，修改 Spring Boot 自动配置的默认值。
# YAML 简介
YAML 全称`YAML Ain't Markup Language`，它是一种以数据为中心的标记语言，比 XML 和 JSON 更适合作为配置文件。

想要使用 YAML 作为属性配置文件（以`.yml`或`.yaml`结尾），需要将 SnakeYAML 库添加到`classpath`下，Spring Boot 中的`spring-boot-starter-web`或`spring-boot-starter`都对 SnakeYAML 库做了集成， 只要项目中引用了这两个`Starter`中的任何一个，Spring Boot 会自动添加 SnakeYAML 库到`classpath`下。
# YAML 语法
YAML 的语法如下：
* 使用缩进表示层级关系。
* 缩进时不允许使用 Tab 键，只允许使用空格。
* 缩进的空格数不重要，但同级元素必须左侧对齐。
* 大小写敏感。

```
spring:
  profiles: dev
  datasource:
    url: jdbc:mysql://127.0.01/banchengbang_springboot
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver
```
# YAML 常用写法
YAML 支持以下三种数据结构：
* 对象：键值对的集合
* 数组：一组按次序排列的值
* 字面量：单个的、不可拆分的值

## YAML 字面量写法
字面量是指单个的，不可拆分的值，例如：数字、字符串、布尔值、以及日期等。

在 YAML 中，使用`key:[空格]value`的形式表示一对键值对（空格不能省略），如`url: www.biancheng.net`。

字面量直接写在键值对的`value`中即可，且默认情况下字符串是不需要使用单引号或双引号的。
```
name: bianchengbang
```
若字符串使用单引号，则会转义特殊字符。
```
name: zhangsan \n lisi
```
输出结果为：
```
zhangsan \n lisi
```
若字符串使用双引号，则不会转义特殊字符，特殊字符会输出为其本身想表达的含义
```
name: zhangsan \n lisi
```
输出结果为：
```
zhangsan 
lisi
```
## YAML 对象写法
在 YAML 中，对象可能包含多个属性，每一个属性都是一对键值对。

YAML 为对象提供了 2 种写法：
* 普通写法，使用缩进表示对象与属性的层级关系。
```
website: 
  name: bianchengbang
  url: www.biancheng.net
```
* 行内写法：
```
website: {name: bianchengbang,url: www.biancheng.net}
```

## YAML 数组写法
YAML 使用“-”表示数组中的元素，普通写法如下：
```
pets:
  -dog
  -cat
  -pig
```
行内写法
```
pets: [dog,cat,pig]
```
## 复合结构
以上三种数据结构可以任意组合使用，以实现不同的用户需求。
```
person:
  name: zhangsan
  age: 30
  pets:
    -dog
    -cat
    -pig
  car:
    name: QQ
  child:
    name: zhangxiaosan
    age: 2
```
# YAML 组织结构
一个 YAML 文件可以由一个或多个文档组成，文档之间使用“---”作为分隔符，且个文档相互独立，互不干扰。如果 YAML 文件只包含一个文档，则“---”分隔符可以省略。
```
---
website:
  name: bianchengbang
  url: www.biancheng.net
---
website: {name: bianchengbang,url: www.biancheng.net}
pets:
  -dog
  -cat
  -pig
---
pets: [dog,cat,pig]
name: "zhangsan \n lisi"
---
name: 'zhangsan \n lisi'
```