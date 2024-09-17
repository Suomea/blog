队列接口：
```java
package java.util;

public interface Queue<E> extends Collection<E> {

	// 添加数据到队尾，成功返回 ture
	// 否则抛出异常
    boolean add(E e);

	// 添加数据到队尾，成功返回 ture，失败返回 false
    boolean offer(E e);

	// 移除并返回队头，队列为空时抛出异常
    E remove();

	// 移除并返回队头，队列为空时返回 null
    E poll();

	// 获取队头，队列为空时抛出异常
    E element();

	// 获取队头，队列为空时返回 null
    E peek();
}
```