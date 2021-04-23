---
title: scala基础学习(和java对比)
categories:
  - Other
tags:
  - Other
date: 2021-04-20 14:18:23
---

### 目录
* [1.目的](#1-目的)
* [2.具体实施](#2-具体实施)
    * [2.1 事前准备](#2-1事前准备)
    * [2.2 实施并验证](#2-2实施并验证)
* [3.为什么采用以上方式](#3-为什么采用以上方式)



## 1.目的

> 没戏，但就是干！

## 2.具体实施



### 2.1事前准备



### 2.2实施并验证

|说明  | Scala |Java | 
| :----------:| :-----------:| :-----------:| 
| 句末结尾   | 可以不用分号 | 必须分号结尾 |
| 交互式编程  | 支持   | 不支持  |
| 先编译在运行 | 支持 | 不支持  |
| 大小写 | 敏感 | 敏感  |
| int等原生类型 | 没有 | 有  |
| 多行字符串 | """ ... """ | 没有，需要转义双引号  |
| for | for(a <- 1 to 3; b <- 1 to 3) | for(int a=1;a<=3;a++){for(int b=1;b<=3;b++){}}  |
| 方法定义 | def fun(a:Int) : Unit ={} | public void fun(int a){} |
| 函数简写 | val fun = (i:Int) => i*2 | 没有区别函数和方法  |
| 格式化输出 | printf("%f,%d") |   |
| main函数 |  |   |
| 数组 | val z:Array[String] = new Array[String](3) | String strArray = new String[3]  |
| 多维数组 | val myMatrix = Array.ofDim[Int](3,3) | int [][] intMatrix = new int[3][3]  |
| 集合-List | val nums: List[Int] = List(1,2,3,4) | List<T> list = new xxxList<>()  |
| 申明类 | class Point(xc:Int){} | class Point{}  |
| 继承类 | class Location(override val xc:Int) extends Point(xc){} | class Location extends Point  |
| 单例 | object | 构造函数private，自身包含一个实例，比如enum  |
| 伴生对象 | object名称与类名共享一个名称，伴生对象中的成员会被当作静态成员处理 | 一个类即可有实例成员和静态成员 |
| 多继承 | trait 可以被多继承，功能类似java中虚拟类，但可以实例化 | 不能多继承  |
| 模式匹配 | def matchTest(x:Int):String=x match{case 1 => "one" case _ => "many"} | switch case break default  |
| 正则表达式 | import scala.util.matching.Regex "1" findFirstIn("212") | Pattern pattern,Matcher  |
| 异常处理 | try{}catch{case ex:Exception=>{}}finally{} | try{}catch(Exception e){}finally{}  |
| 提取器 |  |   |
|  |  |   |
|  |  |   |
|  |  |   |
|  |  |   |


## 3.为什么采用以上方式

















