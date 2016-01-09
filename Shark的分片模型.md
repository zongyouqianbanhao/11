##### kratos支持2类4种分片算法：
- 库内分片类型：
  - 片名连续的库内分片算法；
  - 非片名连续的库内分片算法；
- 一库一片类型：
  - 片名连续的一库一片算法；
  - 非片名连续的一库一片算法；

## 片名连续的库内分片算法：
假设dbSize是32，tbSize是1024，那么每一个库的片数为1024/32=32，而片名是按照0000-1023进行分布的，这就意味着片名是全局唯一的，不允许出现重复。 
算法配置：
```Xml
<property name="dbRuleArray" value="#userinfo_test_id|email_hash# % 1024 / 32" />
<property name="tbRuleArray" value="#userinfo_test_id|email_hash# % 1024 % 32" />
```
![](http://dl.iteye.com/upload/picture/pic/133891/f7d547a8-6d8e-3404-a3f0-6dce8b25042a.jpg)
