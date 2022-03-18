`org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton(java.lang.String, boolean)`

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
	   /**
        * 第一步:我们尝试去一级缓存(单例缓存池中去获取对象,一般情况从该map中获取的对象是直接可以使用的)
        * IOC容器初始化加载单实例bean的时候第一次进来的时候 该map中一般返回空
        */
        Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
        /**
          * 尝试去二级缓存中获取对象(二级缓存中的对象是一个早期对象)
          * 何为早期对象:就是bean刚刚调用了构造方法，还来不及给bean的属性进行赋值的对象(纯净态)
          * 就是早期对象
          */
				singletonObject = this.earlySingletonObjects.get(beanName);
        /**
          * 二级缓存中也没有获取到对象,allowEarlyReference为true(参数是有上一个方法传递进来的true)
          */
				if (singletonObject == null && allowEarlyReference) {
            /**
             * 直接从三级缓存中获取 ObjectFactory对象 这个对接就是用来解决循环依赖的关键所在
             * 在ioc后期的过程中,当bean调用了构造方法的时候,把早期对象包裹成一个ObjectFactory
             * 暴露到三级缓存中
             */
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
               /**
                * 在这里通过暴露的ObjectFactory 包装对象中,通过调用他的getObject()来获取我们的早期对象
                * 在这个环节中会调用到 getEarlyBeanReference()来进行后置处理
                */
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
                         //把早期对象放置在二级缓存,
						this.earlySingletonObjects.put(beanName, singletonObject);
                        //ObjectFactory 包装对象从三级缓存中删除掉
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
}
```

