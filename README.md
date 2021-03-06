## 1. pip database_auto_bulk_operation

```
强大的自动批量聚合操作各种数据库，不需要再调用处手动去喂给批量调用方法一个个组装好了数组。
自动聚合未来未知时间的离散任务成批量任务。

自动批量聚合操作数据库能减少客户端和数据库服务端的io往返次数，效率提升大。

包括支持mysql mongo redis elastic。

主要原理是批量操作对象内部有一个while 1的守护线程，不断的去自动组合列表任务，
然后调用各种数据库的的python包的批量操作方法。
同时加入一个atexit的钩子，防止守护线程随程序一起结束时候，掉一批还未批量插入的尾部任务。


各种数据库的更简单的批次操作，主要是不用再调用处手动切割分组来调用原生中间件操作类的批量操作方法
对于未知时间的离散任务能够自动批量聚合，这种情况下无法自己提前切割分组，使用此包的方式非常适合。

```


## 此包解决的真实场景举例，举一个不用此包造成的写代码重复的后果。

```
如果不使用自动批量聚合会造成到处重复写很多切割分组的代码，思维不缜密的对非整除批次的分组还会漏掉最后一部分余数的批次

举个真实例子，以下是实现每批次插入1000个到mongo，自己要维护一个数组变量mongo_list，
然后每次插入后清除数组，为了防止非整除，还要再在for循环以外插入一次。

临时写一次两次还好，一个项目中写个几十次这样的类似重复逻辑场景的代码，就要吐血了。
写代码的出发点是宁愿十分困难的搞一次，也不愿意不困难但是重复的麻烦几百次。

mongo_list = []
for p_id, value in place_brand_dict.items():
    if not hotel_city.find_one({'_id': p_id}):
        continue
    val_list = []
    for brand_item in value.keys():
        value[brand_item]['star'] = star_dict[brand_item]['star']
        val_list.append(value[brand_item])
    mongo_list.append(UpdateOne({'_id': p_id}, {'$set': {'brands': val_list, 'place_id': p_id}}, upsert=True))
    if len(mongo_list) == 1000:
        city_brand_new.bulk_write(mongo_list)
        mongo_list = []
if mongo_list:
    city_brand_new.bulk_write(mongo_list)
```

## 上面这种代码是固定的从一个已知确定的字典中，把任务写到mongo，这样实现不难，但对未来未知时间的离散任务要批量插入就难了。

```
例如是你写了一个函数是插入mongo记录的，这个函数是被供给被别的地方调用，并不知道会什么时候调用和调用多少次你的mongo插入函数
这种情况下做数组预切割，然后调用pymongo的bulk_write方法，那就不可能实现了。

对于这种场景是最适合使用此包的自动批量聚合功能的。
```