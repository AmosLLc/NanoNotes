# 阿里妈妈Java一面菜鸡经

作者：小错。
https://www.nowcoder.com/discuss/368546来源：牛客网

- JVM运行时区域，本地方法栈干啥的？ 

-  堆，推荐从gcroots到安全点到回收算法（GC）到收集器  

- final，语法上用处和JMM语义相关，final修饰的引用可以指向null吗（可以，但是就不能修改了，null是任何引用类型的默认值，不严格的说是所有object类型的默认值）  

- 抽象类和接口（尽量讲讲自己的实践）  

- 为啥JDK8之前的接口不加default方法？（因为多接口有相同的方法时会冲突，类似C++的多继承问题，JDK8解决：https://blog.csdn.net/weixin_34144848/article/details/92419772，这里能回答出来是阿里的老哥引导的，吹爆）  

- hashmap、concurrenthashmap和hashtable的区别（害，没有深入问我，好像第一面就是问问广度，后面应该就会往死里锤我了）  

- IOC、AOP整一个  

- 动态代理讲一讲（jdk的、cglib的）  

- cglib可以为抽象类生成代理不  





# 阿里妈妈Java菜鸡经-二面

作者：小错。
https://www.nowcoder.com/discuss/369267来源：牛客网

 	
 

 	动态代理有哪些实现方式，区别？ 

 	nio？ 

 	设计模式？ 

 	高并发下如何删减库存？（阿里的老哥引导了我一波，点赞） 

 	synchronized和lock？ 

 	redis数据结构？ 

 	sso单点登录？ 

 	servlet单实例多线程还是多实例多线程？ 

 	幂等? 

 	CAP? 

 	选举算法? 

 	讲讲你项目里你觉得有难度的一个挑战？