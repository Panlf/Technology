# Java秒杀系统

1、redis的预减

预先将需要秒杀的商品加载到Redis中，然后使用Redis进行预减内存

```

// 这边需要使用分布式锁
Long stock = redisTemplate.opsForValue().decrement("seckillGoods:"+goodsId)

if(stock < 0){
	//库存清空				
	redisTemplate.opsForValue().increment("seckillGoods:"+goodsId)
}
//下单

```

2、使用RabbitMQ

```
//通过redis的预减

//将消息发送到rabbitMQ的队列中
MQSender.send("seckillGoods")

```

3、内存标记

```)
//将已经秒杀完的秒杀商品，记录到内存中
if(EmptyStockMap.get(goodsId)){
	return 
}
if(stock < 0){
EmptyStockMap.put(goodsId,true);
	//库存清空				redisTemplate.opsForValue().increment("seckillGoods:"+goodsId);
}

```

4、重复判断

```
//判断库存
//根据goodsId
if(goods.getStockCount() < 1){

}

//判断是否抢购
//根据UserId和goodsId
if(seckillOrder !=null){

}

```

5、分布式锁

```lua
if (redis.call('exists',KEYS[1]) == 1) then
    local stock = tonumber(redis.call('get',KEYS[1]));
    if(stock > 0) then
        redis.call('incrby',KEYS[1],-1)
        return stock;
    end;
    return 0;
end;
```



6、秒杀地址隐藏

```
/{path}/seckill
```

7、验证码

8、限流防刷



