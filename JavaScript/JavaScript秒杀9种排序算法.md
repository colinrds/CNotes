[toc]

## 你必须知道的

1. JS原型
2. 排序中的有序区和无序区
3. 二叉树的基本知识

如果你不知道上面三个东西，还是去复习一下吧，否则，看下面的东西有点吃力。

## 封装丑陋的原型方法

```
Function.prototype.method = function(name, func){
    this.prototype[name] = func;
    return this;
};
```

在下面的排序中都要用到这个方法。

## 9种排序算法的思路和实现

### 插入排序

基本思路：

从无序区的第一个元素开始和它前面有序区的元素进行比较，如果比前面的元素小，那么前面的元素向后移动，否则就将此元素插入到相应的位置。

```
Array.method('insertSort', function(){
    var len = this.length,
        i, j, tmp;
    for(i=1; i<len; i++){
        tmp = this[i];
        j = i - 1;
        while(j>=0 && tmp < this[j]){
            this[j+1] = this[j];
            j--;
        }
        this[j+1] = tmp;
    }
    return this;
});
```

### 二分插入排序

二分插入排序思路：

先在有序区通过二分查找的方法找到移动元素的起始位置，然后通过这个起始位置将后面所有的元素后移。

```
Array.method('bInsertSort', function(){
    var len = this.length,
        i, j, tmp, low, high, mid;
    for(i=1; i<len; i++){
        tmp = this[i];
        low = 0;
        high = i - 1;
        while(low <= high){
            mid = (low+high)/2;
            if(tmp < this[mid]) high = mid - 1;
            else low = mid + 1;
        }
        for(j=i-1; j>=high+1; j--){
            this[j+1] = this[j];            
        }
        this[j+1] = tmp;
    }
    return this;
});
```

### 希尔排序

希尔排序思路:

我们在第 i 次时取gap = n/(2的i次方)，然后将数组分为gap组(从下标0开始，每相邻的gap个元素为一组)，接下来我们对每一组进行直接插入排序。

```
Array.method('shellSort', function(){
    var len = this.length, gap = parseInt(len/2), 
        i, j, tmp;
    while(gap > 0){
        for(i=gap; i<len; i++){
            tmp = this[i];
            j = i - gap;
            while(j>=0 && tmp < this[j]){
                this[j+gap] = this[j];
                j = j - gap;
            }
            this[j + gap] = tmp;
        }
        gap = parseInt(gap/2);
    }
    return this;
});
```

### 冒泡排序

冒泡排序思想:

通过在无序区的相邻元素的比较和替换，使较小的元素浮到最上面。

```
Array.method('bubbleSort', function(){
    var len = this.lenght,
        i, j, tmp;
    for(i=0; i<len; i++){
        for(j=len-1; j>i; j--){
            if(this[j] > this[j-1]){
                tmp = this[j-1];
                this[j-1] = this[j];
                this[j] = tmp;
            }
        }
    }
    return this;
});
```

### 改进的冒泡排序

基本思路：

如果在某次的排序中没有出现交换的情况，那么说明在无序的元素现在已经是有序了，就可以直接返回了。

```
Array.method('rBubbleSort', function(){
    var len = this.length,
        i, j, tmp, exchange;
    for(i=0; i<len; i++){
        exchange = 0;
        for(j=len-1; j>i; j--){
            if(this[j] < this[j-1]){
                tmp = this[j];
                this[j] = this[j-1];
                this[j-1] = tmp;
                exchange = 1;
            }
        }
        if(!exchange) return this;
    }
    return this;
});
```

### 快速排序

快速排序思路：

1. 假设第一个元素为基准元素 
2. 把所有比基准元素小的记录放置在前一部分，把所有比基准元素大的记录放置在后一部分，并把基准元素放在这两部分的中间(i=j的位置)

```
Array.method('quickSort', function(s, t){
    var i=s, j=t,
        tmp;
    if(s < t){
        tmp = this[s];
        while(i!=j){
            while(j>i && this[j]>tmp) j--;//右—>左
            R[i] = R[j];
            while(i<j && this[j]<tmp) i++;//左—>右
            R[j] = R[i]; 
        }
        R[i] = tmp;
        this.quickSort(s, i-1);
        this.quickSort(i+1, t);
    }
    return this;
});
```

### 选择排序

选择排序思路：

在无序区中选出最小的元素，然后将它和无序区的第一个元素交换位置。

```
Array.method('selectSort', function(){
    var len = this.length,
        i, j, k, tmp;
    for(i=0; i<len; i++){
        k = i;
        for(j=i+1; j<len; j++){
            if(this[j] < this[k]) k = j;
        }
        if(k!=i){
            tmp = this[k];
            this[k] = this[i];
            this[i] = tmp;
        }
    }
    return this;
});
```

### 堆排序

堆排序是一种树形选择排序方法(注意下标是从1开始的，也就是R[1...n])。

堆排序思路：

1. 初始堆：

将原始数组调整成大根堆的方法——筛选算法：

比较R[2i]、R[2i+1]和R[i]，将最大者放在R[i]的位置上(递归调用此方法到结束)

2. 堆排序：

每次将堆顶元素与数组最后面的且没有被置换的元素互换。

```
Array.method('createHeap', function(low, high){
    var i=low, j=2*i, tmp=this[i];
    while(j<=high){
        if(j< high && this[j]<this[j+1]) j++; //从左右子节点中选出较大的节点
        if(tmp < this[j]){                    //根节点(tmp)<较大的节点
            this[i] = this[j];
            i = j;
            j = 2*i;
        }else break;
    }
    this[i] = tmp;                            //被筛选的元素放在最终的位置上
    return this;
});

Array.method('heapSort', function(){
    var i, tmp, len=this.length-1;
    for(i=parseInt(len/2); i>=1; i--) this.createHeap(i, len);
    for(i=len; i>=2; i--){
        tmp = this[1];
        this[1] = this[i];
        this[i] = tmp;
        this.createHeap(1, i-1);
    }
    return this;
});
```

### 归并排序

归并排序思路：

1. 归并

从两个有序表R[low...mid]和R[mid+1...high]，每次从左边依次取出一个数进行比较，将较小者放入tmp数组中，最后将两段中剩下的部分直接复制到tmp中。

这样tmp是一个有序表，再将它复制加R中。(其中要考虑最后一个子表的长度不足length的情况)

2. 排序

自底向上的归并，第一回：length=1；第二回：length=2*length ...

```
Array.method('merge', function(low, mid, high){
    var tmp = new Array(), i = low, j=mid+1, k=0;
    while(i<=mid && j<=high){
        if(this[i] <= this[j]){//比较第一部分和第二部分，取较小者
            tmp[k] = this[i];
            i++;
            k++;
        }else{
            tmp[k] = this[j];
            j++;
            k++;
        }
    }
    while(i<=mid){
        tmp[k] = this[i];
        i++;
        k++;
    }
    while(j<=high){
        tmp[k] = this[j];
        j++;
        k++;
    }
    for(k=0,i=low; i<=high; k++,i++) this[i] = tmp[k];

    return this;
});
Array.method('mergePass', function(length, n){
    var i;
    for(i=0; i+2*length-1<n; i=i+2*length) this.merge(i, i+length-1, i+2*length-1);
    if(i+length-1 < n) this.merge(i, i+length-1, n-1); //考虑到最后一个子表的长度可能小于length，所以要特殊处理一下

    return this;
});

Array.method('mergeSort', function(){
    var len = this.length,
        length;
    for(length=1; length<len; length=2*length) this.mergePass(length, len);

    return this;
});
```

## 测试

```
var out = function(){
    console.log(arguments);
}
var test = [0,1,7,6,3,4,9,8];
out(test.insertSort(), 'insertSort');
var test = [0,1,7,6,3,4,9,8];
out(test.bInsertSort(), 'bInsertSort');
var test = [0,1,7,6,3,4,9,8];
out(test.shellSort(), 'shellSort');
var test = [0,1,7,6,3,4,9,8];
out(test.bubbleSort(), 'bubbleSort');
var test = [0,1,7,6,3,4,9,8];
out(test.rBubbleSort(), 'rBubbleSort');
var test = [0,1,7,6,3,4,9,8];
out(test.quickSort(0, test.length-1), 'quickSort');
var test = [0,1,7,6,3,4,9,8];
out(test.selectSort(), 'selectSort');
var test = [0,1,7,6,3,4,9,8];
out(test.heapSort(), 'heapSort');
var test = [0,1,7,6,3,4,9,8];
out(test.mergeSort(), 'mergeSort');
```