##### com.aicoin.job.component

######  BitmainBLC10ComponentJob类 

实现StatefulJob接口，核心方法为重写的excute方法

```java
@Override
public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException 
{
    ...
}
```

该方法先发送get请求获取json格式数据。将数据反序列化为BLC10ComponentEntity类的实例，把实例的属性存储到redis数据库。

##### com.aicoin.job.task

###### BitmexTask类

继承自BaseJob，定义了该类的核心方法statisticBitmexCoin()

```java
@Async
    public void statisticBitmexCoin(String symbol)
    {
        ...
    }
```

该方法从www.bitmex.com上获取比特币交易的开盘收盘最低最高数值发送给kafka。