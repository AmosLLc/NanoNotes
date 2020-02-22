[TOC]

### PriorityQueue

#### 概述

- 优先级队列，基于堆实现。
- 堆是一种数据结构，**概念上是==树==，存储上是==数组==**，父子有特殊顺序，根是最大/最小值，构建、添加、删除效率都很高。
- 实现了队列接口 Queue，每个元素都有优先级，**队头**的元素永远都是**优先级最高**的。
- 内部元素**不是完全有序**，但是**逐个**出队列会得到**完全有序**的输出。
- 最先出队的总是优先级最高的，即排序中的第一个。
- 查看头部元素效率很高，为O(1)， 入队、出队效率比较高，为O(log2(N))，构建堆 heapify 的效率为 O(N)。



#### 应用实例

##### ==TopK== 问题

用 PriorityQueue 默认是自然顺序排序，**要选择最大的k个数，构造最小堆**，每次取数组中剩余数与堆顶的元素进行比较，如果新数比堆顶元素大，则删除堆顶元素，并添加这个新数到堆中。可用于元素个数不确定还需要实时**动态添加**的问题。也可以求**第 K 个**元素。

Java中的PriorityQueue来实现堆，用PriorityQueue的实现的代码如下：

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



##### ==求中值==

求中值的一个基本思路是排序，排序之后找到中间值。如果元素会动态添加，就可以使用堆。维护两个堆，一个最大堆，一个最小堆。

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
        this.maxP = new PriorityQueue<>(11,Collections.<E>reverseOrder());
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









