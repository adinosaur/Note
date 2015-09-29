寻找第k小的数
=========
寻找一个给定序列中的第k小的问题，又称为*选择问题*。
诚然，可以在n * lgk的时间内完成，比如维护一个大小为k的堆（小根堆），然后遍历整个序列。

##1. 选择算法的一个特例——Min-Max选择算法

###基本说明
Min-Max选择算法是找出序列中最小和最大的元素。

###问题描述
1. 问题1
问：找出n个元素中的min需要多少次比较？
答：n-1次

2. 问题2
问：找出n个元素中的max需要多少次比较？
答：还是n-1次

3. 问题3
问：找出n个元素的max和min需要多少次比较？

对于这个问题，2n-2次是肯定的，但是还能更少。

###解决方法
方法是，维护一对min，max分别表示当前的最小值和最大值。接下来遍历，每次取出两个元素，这两个元素先比较一遍，较大的元素和max比较，较小的元素和min比较。
这样，每2个元素需要3次比较，因而n个元素需要3 * (n / 2)次比较。

##2. 期望时间为线性的选择算法——randomized-select

###基本说明
该算法名为randomized-select，能够在θ(n)的期望时间内找出序列中第k小的数。
显然该算法是一个随机化算法，因而计算时间复杂度是计算复杂度的期望值。

###解决方法
利用了随机化quick-sort算法中的randomized-partition算法。
randomized-partition算法随机地选取一个数作为主元，并在O(n)的时间内将序列分成两部分，比主元小的元素排在前面，其余的排在后面，并返回主元所在的下标。

###代码实现
在借用了randomized-partition算法后，randomized-select算法可以如下实现：

```
int randomized_select(vector<int>& v, int b, int e, int k)
{
    if (b == e)
        return v[b];

    int q = randomized_partition(v, b, e);
    int i = q - b + 1;

    if (i == k)
        return A[q];
    else if (k < i)
        return randomized_select(v, b, q-1, k);
    else
        return randomized_select(v, q+1, e, k-i);}
```
该算法期望时间是线性的，最坏时间是O(n^2)，当每次随机地选取主元都不幸地选到最小的或最大的是最坏情况。

##3. 最坏时间为线性的选择算法——select算法

###基本说明
*详细参考《算法导论 第3版》123页*
select算法与randomized-select算法比较类似，不同之处在于用不同的方式选择主元来执行partition算法。

###问题描述
randomized-select算法使用的是randomized-partition算法随机地选取一个元素作为主元然后执行partition。
这种方式选择主元带来的问题是，有可能会选到一个很差的主元使得n个元素的序列被分为1：n-1。
此时最坏情况下randomized-select算法在一轮迭代时耗费O(n)的时间（partition算法的执行时间为O(n)）仅将问题的规模降低1个单位。因此最坏情况下randomized-select算法的时间复杂度是O(n ^ 2)。

###解决方法
select算法应对上述问题是通过做多一些工作，保证了每一次select的迭代对问题规模的缩减都控制在一定的范围内。
通过这个保证即可以算法的最坏时间为O(n)。

select算法的描述如下：

1. 将输入数组的n个元素分为5个一组，其中最后一组可能不足5个，因此总共有n/5组。
2. 对每个分组进行插入排序（插入排序的元素较少时，速度较快），然后找出每组的中位数。（其中这一步的时间复杂度是O(n)，解释是对大小为O(1)的集合调用O(n)次的插入排序）
3. 对第2步找出的n/5个的中位数，递归地调用select算法，找出其中的中位数x。（时间复杂度为原问题规模的1/5）
4. 用第3步得到的x值对输入数组执行partition操作。

之后的便和randomized-select算法的处理一样了。











