# Spring Event使用注意点

`Spring`广播消息时，`Spring`会在 `ApplicationContext` 中查找所有的监听者，即需要 `getBean` 获取 `bean` 实例。然而 `Spring` 有个限制 - `ApplicationContext` 关闭期间，不得`GetBean` 否则会报错。

使用 `SpringEvent` 之前，一定要先治理服务，确保服务关闭时，先切断入口流量（Http、MQ、RPC），然后再关闭服务，关闭 `Spring` 上下文！

`SpringBoot` 会在`Spring`完全启动完成后，才开启`Http`流量。这给了我们启示：应该在`Spring`启动完成后开启入口流量。`Rpc`和 `MQ`流量 也应该如此，所以建议大家 在 `SmartLifecype` 或者 `ContextRefreshedEvent` 等位置 注册服务，开启流量。