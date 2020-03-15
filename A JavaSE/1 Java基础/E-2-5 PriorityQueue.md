[TOC]

### PriorityQueue

#### 1 概述

- 优先级队列，基于堆实现。
- 堆是一种数据结构，**概念上是==树==，存储上是==数组==**，父子有特殊顺序，根是最大/最小值，构建、添加、删除效率都很高。
- 实现了队列接口 Queue，每个元素都有优先级，**队头**的元素永远都是**优先级最高**的。
- 内部元素**不是完全有序**，但是**逐个**出队列会得到**完全有序**的输出。
- 最先出队的总是优先级最高的，即排序中的第一个。
- 查看头部元素效率很高，为O(1)， 入队、出队效率比较高，为O(log2(N))，构建堆 heapify 的效率为 O(N)。
- PriorityQueue 是一种无界的，线程**不安全**的队列。



#### 2 基本使用

PriorityQueue 使用跟普通队列一样，唯一区别是 PriorityQueue 会根据**排序规则**决定谁在队头，谁在队尾。

使用方式如下

```java
//自定义比较器，降序排列
static Comparator<Integer> cmp = new Comparator<Integer>() {
    public int compare(Integer e1, Integer e2) {
        return e2 - e1;
    }
};
public static void main(String[] args) {
    // 不用比较器，默认升序排列
    Queue<Integer> q = new PriorityQueue<>();
    q.add(3);
    q.add(2);
    q.add(4);
    while(!q.isEmpty()) {
        System.out.print(q.poll()+" "); // 2 3 4 
    }

    // 使用自定义比较器，降序排列
    Queue<Integer> qq = new PriorityQueue<>(cmp);
    qq.add(3);
    qq.add(2);
    qq.add(4);
    while(!qq.isEmpty()) {
        System.out.print(qq.poll()+" "); // 4 3 2 
    }
}
```



#### 3 源码分析

基本属性如下。

```java
public class PriorityQueue<E> extends AbstractQueue<E>
    implements java.io.Serializable {
    transient Object[] queue;    //队列容器， 默认是11
    private int size = 0;  //队列长度
    private final Comparator<? super E> comparator;  //队列比较器， 为null使用自然排序
    //....
}
```

##### ① 入队列

```java
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);      // 当队列长度大于等于容量值时，自动拓展
    size = i + 1;
    if (i == 0)
        queue[0] = e;
    else
        siftUp(i, e); // 上浮
    return true;
}

private void siftUp(int k, E x) {
    if (comparator != null)
        siftUpUsingComparator(k, x);   // 指定比较器
    else
        siftUpComparable(k, x);   // 没有指定比较器，使用默认的自然比较器
}

private void siftUpComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>) x;
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (key.compareTo((E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = key;
}

private void siftUpUsingComparator(int k, E x) {
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (comparator.compare(x, (E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = x;
}
```

这里简单比较，使用选择排序法将入列的元素放**左边或者右边。**
从源码上看 PriorityQueue 的入列操作并**没有**对所有加入的元素**进行优先级排序**。**仅仅保证数组==第一个==元素是最小的即可。**



##### ② 出队列

```java
// 出队列
public E poll() {
    if (size == 0)
        return null;
    int s = --size;
    modCount++;
    // 直接取队列第一个出队列
    E result = (E) queue[0];
    E x = (E) queue[s];
    queue[s] = null;
    if (s != 0)
        // 寻找剩下最小的放到首位
        siftDown(0, x);
    return result;
}

private void siftDown(int k, E x) {
    if (comparator != null)
        siftDownUsingComparator(k, x);  // 指定比较器
    else
        siftDownComparable(k, x);       // 默认比较器
}

private void siftDownComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>)x;
    int half = size >>> 1;        // loop while a non-leaf
    while (k < half) {
        int child = (k << 1) + 1; // assume left child is least
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
            c = queue[child = right];
        if (key.compareTo((E) c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = key;
}

@SuppressWarnings("unchecked")
private void siftDownUsingComparator(int k, E x) {
    int half = size >>> 1;
    while (k < half) {
        int child = (k << 1) + 1;
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            comparator.compare((E) c, (E) queue[right]) > 0)
            c = queue[child = right];
        if (comparator.compare(x, (E) c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = x;
}

```

上面源码，当第一个元素出列之后，对剩下的元素进行再排序，**挑选出最小的元素排在数组第一个位置。**

通过上面源码，也可看出 PriorityQueue 并**不是线程安全队列**，因为 offer/poll 都没有对队列进行**锁定**，所以，如果要拥有线程安全的优先级队列，需要额外进行加锁操作。





#### 4 应用实例

##### ① ==TopK== 问题

用 PriorityQueue 默认是自然顺序排序，**要选择最大的k个数，构造最小堆**，每次取数组中剩余数与堆顶的元素进行比较，如果新数比堆顶元素大，则删除堆顶元素，并添加这个新数到堆中。可用于元素个数不确定还需要实时**动态添加**的问题。也可以求**第 K 个**元素。

Java 中的 PriorityQueue 来实现堆，用 PriorityQueue 的实现的代码如下：

```java
public class findTopK {
    // 找出前k个最大数，采用小顶堆实现
    public static int[] findKMax(int[] nums, int k) {
        // 队列默认自然顺序排列，小顶堆，不必重写compare
        PriorityQueue<Integer> pq = new PriorityQueue<>(k);

        for (int num : nums) {
            if (pq.size() < k) {
                pq.offer(num);
            } else if (pq.peek() < num) { // 如果堆顶元素 < 新数，则删除堆顶，加入新数入堆
                pq.poll();
                pq.offer(num);
            }
        }

        int[] result = new int[k];
        for (int i = 0; i < k && !pq.isEmpty(); i++) {
            result[i] = pq.poll();
        }
        return result;
    }

    public static void main(String[] args) {
        int[]arr=new int[]{1, 6, 2, 3, 5, 4, 8, 7, 9};
        System.out.println(Arrays.toString(findKMax(arr,5)));
    }
}
/**
* 输出：[5, 6, 7, 8, 9]
*/
```



##### ② ==求中值==

求中值的一个基本思路是排序，排序之后找到中间值。如果元素会**动态添加**，就可以使用堆。**维护两个堆，一个最大堆，一个最小堆。**

思路如下：

- 设当前中位数为 m, 最大堆维护的是 ≤ m 的元素，最小堆维护的是 ≥ m 的元素，但两个堆都不包含 m。
- 当新元素到达时，将新元素与 m 进行比较，若新元素 ≤ m， 则将其加入最大堆，否则加入最小堆。
- 第二步后，如果此时最小堆与最大堆的元素个数差值 ≥ 2，则将 m 加入元素个数少的堆中，然后从元素个数多的堆中将根节点移除并幅值给 m 即可。

```java
import java.util.Collection;
import java.util.Collections;
import java.util.PriorityQueue;

public class Midian<E> {
    private PriorityQueue<E> minP;
    private PriorityQueue<E> maxP;
	// 中值
    private E m;
    public Midian(){
        this.minP = new PriorityQueue<>();
        this.maxP = new PriorityQueue<>(11, Collections.<E>reverseOrder());
    }

    private int compare(E e , E m){
        Comparable<? super E>  cmpr = (Comparable<? super E>) e;
        return cmpr.compareTo(m);
    }

    public void add(E e){
		// 第一个元素
        if (m == null){
            m = e;
            return;
        }
		// 小于中值加入最大堆
        if (compare(e , m) <= 0){
            maxP.add(e);
        }else {
            minP.add(e);
        }

        if (minP.size() - maxP.size() >=2){
			// 最小堆元素个数多，将m加入最大堆中，然后最小堆中根移除赋值给m
            maxP.add(this.m);
            this.m = minP.poll();
        }else if (maxP.size() - minP.size() >= 2){
            minP.add(this.m);
            this.m = maxP.poll();
        }
    }

    public void addAll(Collection<? extends E> c){
        for(E e:c){
            add(e);
        }
    }

    public E getM(){
        return m;
    }
}

public static void main(String[] args) {
    Midian<Integer> integerMidian = new Midian<>();
    List<Integer> list = Arrays.asList(new Integer[] {1,2,3,4,5,6,7,8,9,10});
    integerMidian.addAll(list);
    System.out.println(integerMidian.getM());
}
```









