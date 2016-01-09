##### kratos支持2类4种分片算法：
- 库内分片类型：
  - 片名连续的库内分片算法；
  - 非片名连续的库内分片算法；
- 一库一片类型：
  - 片名连续的一库一片算法；
  - 非片名连续的一库一片算法；

## 片名连续的库内分片算法：
假设dbSize是32，tbSize是1024，那么每一个库的片数为1024/32=32，而片名是按照0000-1023进行分布的，这就意味着片名是全局唯一的，不允许出现重复。 

片名连续的库内分片算法配置：
```Xml
<property name="consistent" value="true" />
<property name="dbRuleArray" value="#userinfo_test_id|email_hash# % 1024 / 32" />
<property name="tbRuleArray" value="#userinfo_test_id|email_hash# % 1024 % 32" />
```
![](http://dl.iteye.com/upload/picture/pic/133891/f7d547a8-6d8e-3404-a3f0-6dce8b25042a.jpg)

## 非片名连续的库内分片算法：
假设dbSize是32，tbSize是1024，那么每一个库的片数为1024/32=32，而片名在每一个库中都是按照0-31进行分布的，也就是说，非全局唯一，这样的好处，相对于片名连续更容易扩容和数据迁移。

非片名连续的库内分片算法配置：
```Xml
<property name="consistent" value="false" />
<property name="dbRuleArray" value="#userinfo_test_id|email_hash# % 1024 / 32" />
<property name="tbRuleArray" value="#userinfo_test_id|email_hash# % 1024 % 32" />
```
![](http://dl.iteye.com/upload/picture/pic/133951/572c7b86-5198-3456-8e74-06bc3433e150.jpg)

## 片名连续的一库一片算法：
假设dbSize是1024，tbSize是1024，那么每一个库的片数为1024/1024=1，而片名是按照0000-1023进行分布的， 这就意味着片名是全局唯一的，不允许出现重复。相对于库内分片，一库一片的算法更简单和直观，因为一库一片只跟库相关，与片无关，算出了库，就等于间接算 出了片。

片名连续的一库一片算法配置：
```Xml
<property name="consistent" value="true" />
<property name="dbRuleArray" value="#userinfo_test_id|email_hash# % 16" />
```
![](http://dl.iteye.com/upload/picture/pic/133893/11a51597-bec8-321e-9389-2429bd01ff7e.jpg)

## 非片名连续的一库一片算法：
假设dbSize是1024，tbSize是1024，那么每一个库的片数为1024/1024=1，在此大家需要注意，非片名连续的一库一片算 法，所有片名无需后缀(比如tb_0000)，也就是说，有1024个库，即有1024个片，但片名是全局一致的，没有任何后缀，全局所有的片名都叫做 tab。 

非片名连续的一库一片算法配置：
```Xml
<property name="consistent" value="false" />
<property name="dbRuleArray" value="#userinfo_test_id|email_hash# % 16" />
```
![](http://dl.iteye.com/upload/picture/pic/133953/4fcb6744-daa1-34ef-817c-a412a45a37b3.jpg)