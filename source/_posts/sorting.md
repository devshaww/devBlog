---
title: 排序算法整合
date: 2016-01-07 20:58:40
categories: 数据结构
---

## 选择排序 `N2/2`次比较和`N`次交换
> 每次`内循环`都把当前`最小`的元素选出来，然后通过`外循环`将值交换。
> 无论是`乱序数组`还是`顺序数组`，用的时间，竟然一样长。
> 我们会看到，其他排序方法会更善于利用`输入的初始状态`。
<!--more-->

```
public static void sort(int[] a) {
    int N = a.length;
    for(int i = 0; i < N; i++) {
        int min = i;
        for(int j = i+1; a[j] < a[min]; j++)
            min = j;
        int tmp = a[i];
        a[i] = a[min];
        a[min] = tmp;
    }
}
```

## 直接插入 N2/2次比较和N2/2次交换
> 直接插入的使用方法是给定一个任意序列，用这个函数把它排列出来。
  原理是：将一个记录插入到已排好序的序列中，从而得到一个新的，记录数增一的序列。
  一开始把第一个记录看作一个有序序列，第二个记录拿进来，排好；第三个记录拿进来，排好。这样，每拿一个记录出来，就和之前排好的有序序列里的最后一个元素开始作比较，找到合适的插入位置，原序列该后移后移.

```
public static void sort(int[] a) {
    int N = a.length;
    for(int i = 1; i < N; i++) {
        for(int j = i; a[j] < a[j-1] && j > 0; j--) {
            int tmp = a[j];
            a[j] = a[j-1];
            a[j-1] = tmp;
        }
    }
}

```

## 二分法（折半插入）
> 折半插入的前提是：向一个有序序列里插入元素

```
void BInsertSort(SqList &L) {
    for( i = 2; i <= L.Length; i++) {
        L.r[0] = L.r[i];
        low = 1; high = i - 1;
        while(low <= high) {
            mid = (low + high)/2;
            if( L.r[mid] > L.r[0] ) {
                high = m - 1;
            }
            else {
                low = m + 1;
            }
        }
        for(j = i - 1; j >= high + 1; --j) 
            L,r[j] = L.r[j];
        L.r[high + 1] = L.r[0];
    }

```


## 希尔排序
> 对于大规模乱序数组，插入排序很慢，因为它只会交换相邻的元素，因此元素只能一点一点地移动。
> 希尔排序的思想是使数组中任意间隔为h的元素都是有序的。
> 在进行排序时，如果h很大，就能移动很远的距离。

```
public static void sort(int[] a) {
    int N = a.length;
    int h = 1;
    while(h < N/3)
        h = 3 * h + 1;
        // 这个序列被称为递增序列，至于哪个序列是最好的，还没有定论。
    while(h >= 1) {
        for(int i = h; i < N; i++) {
            for(int j = i; a[j] < a[j-h] && j >= h; j -= h) {
                int tmp = a[j];
                a[j] = a[j-h];
                a[j-h] = tmp;
            }
        }
        h = h / 3;
    }
}

```

## 归并排序
> `递归地`将数组分成两半分别排序，然后再将结果归并。

#### 自顶向下归并 1/2NlgN~NlgN次比较
```
public class Merge {
	private static int[] aux;
	public static void sort(int[] a) {
		aux = new int[a.length];
		sort(a, 0, a.length-1);
	}

	private static void sort(int[] a, int lo, int hi) {
		if(hi <= lo) return;
		int mid = lo + (hi - lo) / 2;
		sort(a, lo, lo + (hi-lo) / 2);
		sort(a, mid + 1, hi);
		merge(a, lo, mid, hi);
	}

	// merge方法将子数组a[lo..mid]和a[mid+1..hi]归并成一个有序的数组并将结果存放在a[lo..hi]中
	public static void merge(int[] a, int lo, int mid, int hi) {
		int i = lo, j = mid + 1;
		for(int k = lo; k <= hi; k++)
			aux[k] = a[k];
		for(int k = lo; k <= hi; k++) {
			if(lo > mid) a[k] = aux[j++]; // 左半边的元素取完则取右半边的元素
			else if(j > hi) a[k] = aux[i++]; // 右半边的元素取完则取左边的元素
			else if(aux[i] > aux[j]) a[k] = aux[j++];
			else a[k] = aux[i++];
		}
	}
}

```
## 快速排序
> 与归并互补: 归并将数组分成两个子数组分别排序;而快排将数组排序的方式则是当`两个子数组都有序时`整个数组也就有序了.
> 快排中有个`partition`的概念：假设`partition`为`a[k]`, 那么a[k]`前面`的元素通通`小于等于`a[k], 同理a[k]`后面`的元素通通`大于等于`a[k].

```
public class Quick {
	private static int partition(int[] a, int lo, int hi) {
		int i = lo, j = hi + 1;  // 左右扫描指针
		int v = a[lo];  // 切分元素
		while(true) {
			while(a[++i] < v) {
				if (i == hi) break;
			}
			while(v < a[--j]) {
				if (j == lo) break;
			}
			if (i >= j) break;
			int tmp = a[i];
			a[i] = a[j];
			a[j] = tmp;
		}
		int tmp = a[lo];
		a[lo] = a[j];
		a[j] = tmp;
		return j;
	}

	public static void sort(int[] a) {
		StdRandom.shuffle(a);
		sort(a, 0, a.length - 1);
	}

	private static void sort(int[] a, int lo, int hi) {
		if (hi <= lo) return;
		int j = partition(a, lo, hi);
		sort(a, lo, j-1);
		sort(a, j+1, hi);
	}
}
```