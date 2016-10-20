## Memcached安装及使用

### Memcached安装

Memcached安装非常简单：

Debian/Ubuntu: apt-get install libevent-dev Redhat/Centos: yum install libevent-devel 

```bash
wget http://memcached.org/latest
tar -zxvf memcached-1.x.x.tar.gz
cd memcached-1.x.x
./configure && make && make test && sudo make install
```

### Memcached客户端使用

满怀热情的看完Memcached及Memcached Client各实现版本，本来想用Memcached做缓存服务器使用的。但具体实用时，发现Memcached做得事情非常有限，归咎于它的设计适用于简单的key - value存储，并没有namespace/group这一说法。只能继续找其他的缓存方案，下面贴出使用例子：

客户端使用xmemcached，并使用simple-spring-memcached的注解形式

1. pom文件配置

	```xml
	<simple.spring.memcached.version>3.6.0</simple.spring.memcached.version>
	....
	<dependency>
		<groupId>com.google.code.simple-spring-memcached</groupId>
		<artifactId>simple-spring-memcached</artifactId>
		<version>${simple.spring.memcached.version}</version>
	</dependency>
	<dependency>
		<groupId>com.google.code.simple-spring-memcached</groupId>
		<artifactId>xmemcached-provider</artifactId>
		<version>${simple.spring.memcached.version}</version>
	</dependency>
	```
2. spring配置文件

	```xml
	<import resource="simplesm-context.xml" />
	
	<bean name="defaultMemcachedClient" class="com.google.code.ssm.CacheFactory">
		<property name="cacheClientFactory">
			<bean class="com.google.code.ssm.providers.xmemcached.MemcacheClientFactoryImpl" />
		</property>
		<property name="addressProvider">
			<bean class="com.google.code.ssm.config.DefaultAddressProvider">
				<property name="address" value="10.0.11.135:11211" />
			</bean>
		</property>
		<property name="configuration">
			<bean class="com.google.code.ssm.providers.CacheConfiguration">
				<!-- 使用一致性HASH算法 -->
				<property name="consistentHashing" value="true" />
				<!-- 使用二进制协议，比文本协议好 -->
				<property name="useBinaryProtocol" value="true" />
			</bean>
		</property>
	</bean>
	```
3. JAVA注解

	```java
	@Override
	@ReadThroughSingleCache(namespace = GlobalConstants.CACHE_KEY_ROLE)
	public Role load(@ParameterValueKeyProvider Integer id) {
		return super.get(id, OpenClose.OPEN, OpenClose.CLOSE);
	}
	```
	
	```java
	//测试代码    ======= 显示Memcached所有的key - value
	public void listCacheObjs() throws Exception {
		final String ip = "10.0.11.135:11211";
		
		MemcachedClientBuilder builder = new XMemcachedClientBuilder(AddrUtil.getAddresses(ip));
		MemcachedClient client = builder.build();
		KeyIterator it = client.getKeyIterator(AddrUtil.getOneAddress(ip));
		while (it.hasNext()) {
			String key = it.next();
			LOG.info("====== Key/value : {0} / {1}", key, client.get(key));
		}
    }
	```
	
使用就是这么简单，但功能也比较单一，适合简单的key-value场景。

> 注意事项：  
> 1.`@ReadThroughSingleCache` 作为一个cache存储，即不管你返回值是List还是int，统统转成字符窜后存储。`@ReadThroughMultiCache` list数据是作为多个cache存储，且方法的参数必须包含list的参数。`@ReadThroughAssignCache` 自定义cache的KEY值。  
> 2. 如果方法的参数为一个Entity对象，则需要在主键的get方法上加上`@CacheKeyMethod`注解，表明此Entity生成key的方法为此主键的get方法。如果对象为联合主键，则自己建一个`public String cacheKey() { ... }`,在此方法上加上`@CacheKeyMethod`就可以了。如果没有`@CacheKeyMethod`注解，则使用此对象的`toString`方法生成KEY。标记`@CacheKeyMethod`注解的方法必须返回`String`。

