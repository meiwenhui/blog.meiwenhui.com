---
layout: post
title: "Mybatis二级缓存中装饰者模式的使用"
description: "数据库的五种隔离级别及spring事务(Transaction)的七种事务传播行为"
keywords: "Mybatis, 装饰者模式"
---

```java
public interface Cache {

	// 取得ID
	String getId();

	// 存入值
	void putObject(Object key, Object value);

	// 获取值
	Object getObject(Object key);

	// 删除值
	Object removeObject(Object key);

	// 清空
	void clear();

	// 取得大小
	int getSize();

	// 取得读写锁, 从3.2.6开始没用了，要SPI自己实现锁
	ReadWriteLock getReadWriteLock();

}
```

```java
/*
 * FIFO缓存 这个类就是维护一个FIFO链表，其他都委托给所包装的cache去做。典型的装饰模式
 */
public class FifoCache implements Cache {

	private final Cache delegate;
	private Deque<Object> keyList;
	private int size;

	public FifoCache(Cache delegate) {
		this.delegate = delegate;
		this.keyList = new LinkedList<Object>();
		this.size = 1024;
	}

	@Override
	public String getId() {
		return delegate.getId();
	}

	@Override
	public int getSize() {
		return delegate.getSize();
	}

	public void setSize(int size) {
		this.size = size;
	}

	@Override
	public void putObject(Object key, Object value) {
		cycleKeyList(key);
		delegate.putObject(key, value);
	}

	@Override
	public Object getObject(Object key) {
		return delegate.getObject(key);
	}

	@Override
	public Object removeObject(Object key) {
		return delegate.removeObject(key);
	}

	@Override
	public void clear() {
		delegate.clear();
		keyList.clear();
	}

	@Override
	public ReadWriteLock getReadWriteLock() {
		return null;
	}

	private void cycleKeyList(Object key) {
		// 增加记录时判断如果记录已超过1024条，会移除链表的第一个元素，从而达到FIFO缓存效果
		keyList.addLast(key);
		if (keyList.size() > size) {
			Object oldestKey = keyList.removeFirst();
			delegate.removeObject(oldestKey);
		}
	}

}
```

```java
/**
 * 日志缓存 添加功能：取缓存时打印命中率
 *
 */
public class LoggingCache implements Cache {

	// 用的mybatis自己的抽象Log
	private Log log;
	private Cache delegate;
	protected int requests = 0;
	protected int hits = 0;

	public LoggingCache(Cache delegate) {
		this.delegate = delegate;
		this.log = LogFactory.getLog(getId());
	}

	@Override
	public String getId() {
		return delegate.getId();
	}

	@Override
	public int getSize() {
		return delegate.getSize();
	}

	@Override
	public void putObject(Object key, Object object) {
		delegate.putObject(key, object);
	}

	// 目的就是getObject时，打印命中率
	@Override
	public Object getObject(Object key) {
		// 访问一次requests加一
		requests++;
		final Object value = delegate.getObject(key);
		// 命中了则hits加一
		if (value != null) {
			hits++;
		}
		if (log.isDebugEnabled()) {
			// 就是打印命中率 hits/requests
			log.debug("Cache Hit Ratio [" + getId() + "]: " + getHitRatio());
		}
		return value;
	}

	@Override
	public Object removeObject(Object key) {
		return delegate.removeObject(key);
	}

	@Override
	public void clear() {
		delegate.clear();
	}

	@Override
	public ReadWriteLock getReadWriteLock() {
		return null;
	}

	@Override
	public int hashCode() {
		return delegate.hashCode();
	}

	@Override
	public boolean equals(Object obj) {
		return delegate.equals(obj);
	}

	private double getHitRatio() {
		return (double) hits / (double) requests;
	}

}
```


```java
/**
 * 序列化缓存 用途是先将对象序列化成2进制，再缓存,好处是将对象压缩了，省内存 坏处是速度慢了
 *
 */
public class SerializedCache implements Cache {

	private Cache delegate;

	public SerializedCache(Cache delegate) {
		this.delegate = delegate;
	}

	@Override
	public String getId() {
		return delegate.getId();
	}

	@Override
	public int getSize() {
		return delegate.getSize();
	}

	@Override
	public void putObject(Object key, Object object) {
		if (object == null || object instanceof Serializable) {
			// 先序列化，再委托被包装者putObject
			delegate.putObject(key, serialize((Serializable) object));
		} else {
			throw new CacheException("SharedCache failed to make a copy of a non-serializable object: " + object);
		}
	}

	@Override
	public Object getObject(Object key) {
		// 先委托被包装者getObject,再反序列化
		Object object = delegate.getObject(key);
		return object == null ? null : deserialize((byte[]) object);
	}

	@Override
	public Object removeObject(Object key) {
		return delegate.removeObject(key);
	}

	@Override
	public void clear() {
		delegate.clear();
	}

	@Override
	public ReadWriteLock getReadWriteLock() {
		return null;
	}

	@Override
	public int hashCode() {
		return delegate.hashCode();
	}

	@Override
	public boolean equals(Object obj) {
		return delegate.equals(obj);
	}

	private byte[] serialize(Serializable value) {
		try {
			// 序列化核心就是ByteArrayOutputStream
			ByteArrayOutputStream bos = new ByteArrayOutputStream();
			ObjectOutputStream oos = new ObjectOutputStream(bos);
			oos.writeObject(value);
			oos.flush();
			oos.close();
			return bos.toByteArray();
		} catch (Exception e) {
			throw new CacheException("Error serializing object.  Cause: " + e, e);
		}
	}

	private Serializable deserialize(byte[] value) {
		Serializable result;
		try {
			// 反序列化核心就是ByteArrayInputStream
			ByteArrayInputStream bis = new ByteArrayInputStream(value);
			ObjectInputStream ois = new CustomObjectInputStream(bis);
			result = (Serializable) ois.readObject();
			ois.close();
		} catch (Exception e) {
			throw new CacheException("Error deserializing object.  Cause: " + e, e);
		}
		return result;
	}

	// 这个Custom不明白何意
	public static class CustomObjectInputStream extends ObjectInputStream {

		public CustomObjectInputStream(InputStream in) throws IOException {
			super(in);
		}

		@Override
		protected Class<?> resolveClass(ObjectStreamClass desc) throws IOException, ClassNotFoundException {
			return Resources.classForName(desc.getName());
		}

	}

}
```


```java
private Cache setStandardDecorators(Cache cache) {
  try {
    MetaObject metaCache = SystemMetaObject.forObject(cache);
    if (size != null && metaCache.hasSetter("size")) {
      metaCache.setValue("size", size);
    }
    if (clearInterval != null) {
      // 刷新缓存间隔,怎么刷新呢，用ScheduledCache来刷，还是装饰者模式，漂亮！
      cache = new ScheduledCache(cache);
      ((ScheduledCache) cache).setClearInterval(clearInterval);
    }
    if (readWrite) {
      // 如果readOnly=false,可读写的缓存 会返回缓存对象的拷贝(通过序列化) 。这会慢一些,但是安全,因此默认是
      // false。
      cache = new SerializedCache(cache);
    }
    // 日志缓存
    cache = new LoggingCache(cache);
    // 同步缓存, 3.2.6以后这个类已经没用了，考虑到Hazelcast, EhCache已经有锁机制了，所以这个锁就画蛇添足了。
    cache = new SynchronizedCache(cache);
    if (blocking) {
      cache = new BlockingCache(cache);
    }
    return cache;
  } catch (Exception e) {
    throw new CacheException("Error building standard cache decorators.  Cause: " + e, e);
  }
}
```
