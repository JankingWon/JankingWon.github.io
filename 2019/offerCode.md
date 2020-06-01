---
title: 面试常见算法题
urlname: offerCode
tags:
  - Offer
  - Code
copyright: true
category: Code
date: 2019-04-21 01:25:35
---

## 全排列

针对1 - n 的数字全排列成字符串

<!--more-->

```JAVA
//全排列
public static void main(String[] args){
        //12 21
        //123 213 132 231 312 321

        //312 321 132 231 123 213
        Scanner scanner = new Scanner(System.in);
        int count = scanner.nextInt();
        ArrayList<String> arrayList = new ArrayList<>();
        loop(arrayList, count);
        System.out.println(arrayList);
}
public static  void loop(ArrayList<String> arrayList, int index){
    if(index == 1){
        arrayList.add("1");
    }else {
        loop(arrayList, index - 1);
        int size = arrayList.size();
        //j表示当前需要扩大的倍数
        for (int j = index - 1; j > 0; j--){
            //i表示之前的大小
            for (int i = size - 1; i >= 0; i--) {
                //添加新元素
                int nowPosition = size * j + i;
                arrayList.add(arrayList.get(i).substring(0, j) + index + arrayList.get(i).substring(j));
            }

        }
        //修改旧元素
        for (int i = size - 1; i >= 0; i--) {
            arrayList.set(i,  index + arrayList.get(i));
        }
    }
}
```

## 排序算法

```JAVA
//插入排序
static void insertSort(int a[], int n) {
    if(n <= 1)
        return;
    for (int i = 1; i < n; i++) {
        for (int j = i - 1; j >= 0 && a[j + 1] < a[j]; j--) {
            int temp = a[j];
            a[j] = a[j + 1];
            a[j + 1] = temp;
        }
    }
}

//冒泡排序
static void bubbleSort(int arr[], int len) {
    int i, j;
    for (i = 0; i < len - 1; i++)
        for (j = 0; j < len - 1 - i; j++)
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
}

//快速排序
static public void quickSort(int[] array, int start, int end) {
    if (start < end) {
        int x = array[start], i = start, j = end;
        while (i < j) {
            while (i < j && array[j] >= x)
                j--;
            if (i < j)
                array[i] = array[j];
            while (i < j && array[i] <= x)
                i++;
            if (i < j)
                array[j] = array[i];
        }
        array[i] = x;
        quickSort(array, start, i - 1);
        quickSort(array, i + 1, end);
    }
}
// 归并排序 （n log n）
public static void sort(int []arr){
    int []temp = new int[arr.length];//在排序前，先建好一个长度等于原数组长度的临时数组，避免递归中频繁开辟空间
    sort(arr,0,arr.length-1,temp);
}
private static void sort(int[] arr,int left,int right,int []temp){
    if(left<right){
        int mid = (left+right)/2;
        sort(arr,left,mid,temp);//左边归并排序，使得左子序列有序
        sort(arr,mid+1,right,temp);//右边归并排序，使得右子序列有序
        merge(arr,left,mid,right,temp);//将两个有序子数组合并操作
    }
}
private static void merge(int[] arr,int left,int mid,int right,int[] temp){
    int i = left;//左序列指针
    int j = mid+1;//右序列指针
    int t = 0;//临时数组指针
    while (i<=mid && j<=right){
        if(arr[i]<=arr[j]){
            temp[t++] = arr[i++];
        }else {
            temp[t++] = arr[j++];
        }
    }
    while(i<=mid){//将左边剩余元素填充进temp中
        temp[t++] = arr[i++];
    }
    while(j<=right){//将右序列剩余元素填充进temp中
        temp[t++] = arr[j++];
    }
    t = 0;
    //将temp中的元素全部拷贝到原数组中
    while(left <= right){
        arr[left++] = temp[t++];
    }
}
//堆排序
public void heapSort(int[] array){
    for (int i = array.length / 2 ; i >= 0 ; i--){ //初始化堆
        heapAdjust(array,i,array.length-1);
    }

    for (int i = array.length-1;i>0;i--){ //排序
        int temp = array[i];
        array[i] = array[0];
        array[0] = temp;

        heapAdjust(array,0,i);
    }
}
private void heapAdjust(int[] array , int parent, int length){
    int temp = array[parent];
    int child = parent*2+1;
    while (child < length){
        if (child+1<length && array[child]<array[child+1] )
            child++;

        if (temp > array[child])
            break;

        array[parent] = array[child];
        parent = child;
        child = 2*child+1;
    }
    array[parent] = temp;
}
```

## LeetCode146. [LRU缓存机制](https://leetcode-cn.com/problems/lru-cache/)

```java
//自己实现

//利用LinkedHashMap
public class LRUCache {
    private LinkedHashMap<Integer, Integer> map;
    private final int CAPACITY;
    public LRUCache(int capacity) {
        CAPACITY = capacity;
        //true表示按照访问顺序排列node
        map = new LinkedHashMap<Integer, Integer>(capacity, 0.75f, true){
            //重写删除元素的触发条件
            protected boolean removeEldestEntry(Map.Entry eldest) {
                return size() > CAPACITY;
            }
        };
    }
    public int get(int key) {
        return map.getOrDefault(key, -1);
    }
    public void put(int key, int value) {
        map.put(key, value);
    }
}
```

###

## 动态规划

### 01背包
```java
//01背包
public static int knapSack(int[] w, int[] v, int C) {
    int size = w.length;
    if (size == 0) {
        return 0;
    }

    int[] dp = new int[C + 1];
    //初始化第一行
    //仅考虑容量为C的背包放第0个物品的情况
    for (int i = 0; i <= C; i++) {
        dp[i] = w[0] <= i ? v[0] : 0;
    }

    for (int i = 1; i < size; i++) {
        for (int j = C; j >= w[i]; j--) {
            dp[j] = Math.max(dp[j], v[i] + dp[j - w[i]]);
        }
    }
    return dp[C];
}

public static void main(String[] args) {
    int[] w = {2, 1, 3, 2};
    int[] v = {12, 10, 20, 15};
    System.out.println(knapSack(w, v, 5));
}
```

### LeetCode416. [分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum/)

```java
//简单易懂的递归
public boolean canPartition(int[] nums) {
        int sum = 0;
        for(int i : nums) {
            sum+=i;
        }
        if(sum%2!=0) return false;
        return helper(nums, nums.length - 1, sum/2);
    }
    private boolean helper(int[] nums, int i, int sum) {
        if(sum == 0) return true;
        else if(i < 0 || sum < 0 || sum < nums[i]) {
            return false;
        }
        else {
            return helper(nums, i - 1, sum - nums[i]) || helper(nums, i - 1, sum);
        }
    }
//效率更高非递归，参考背包问题
	public boolean canPartition(int[] nums) {
        int sum = 0;
        for(int i : nums)
            sum += i;
        //是否偶数
        if(sum % 2 != 0)
            return false;
        int goal = sum / 2;
        //价值为goal的解法
        int itemSize = nums.length;
        int packSize = goal;
        boolean pack[] = new boolean[packSize + 1];
        for(int i = 0; i <= itemSize; i++){
            pack[i] = (i == nums[i]);
        }
        for(int i = 1; i < itemSize; i++){
            for(int j = packSize; j > nums[i]; j--){
                pack[j] = pack[j] || pack[j - nums[i]];
            }
        }
        return pack[packSize];
    }
```



### LeetCode322. [零钱兑换](https://leetcode-cn.com/problems/coin-change/)

```JAVA
public int coinChange(int[] coins, int amount) {
        if(amount == 0)
            return 0;
        int[] dp = new int[amount + 1];
    //初始化从1到amount的情况
        for(int i : coins){
            if(i <= amount)
                dp[i] = 1;
        }
    //从1到amount遍历每一种总额最少需要多少零钱
        for (int i = 1; i <= amount; i++){
            //遍历每一种零钱
            for(int j = 0; j < coins.length; j++){
                if(coins[j] < i && dp[i - coins[j]] != 0){
                    if(dp[i] == 0)
                        dp[i] = dp[i - coins[j]] + 1;
                    else
                        dp[i] = Math.min(dp[i], dp[i - coins[j]] + 1);
                }

            }
        }
        return dp[amount] == 0 ? -1 : dp[amount];
    }
```



### LeetCode377. [组合总和 Ⅳ](https://leetcode-cn.com/problems/combination-sum-iv/)

```java
    public int combinationSum4(int[] nums, int target) {
        int[] dp = new int[target + 1];
        //初始化单个组成的情况
        for(int i : nums){
            if(i <= target)
                dp[i] = 1;
        }
        //遍历每个target
        for(int i = 1; i <= target; i++){
            //遍历每个数
            for(int j = 0; j < nums.length; j++){
                //满足条件则多说明一种情况
                if(i >= nums[j] && dp[i - nums[j]] !=0){
                    dp[i] += dp[i - nums[j]];
                }
            }
        }
        return dp[target];
    }
```

### LeetCode 474. [一和零](https://leetcode-cn.com/problems/ones-and-zeroes/)

**代码超级简洁**，就是不一定能迅速想得出来。

```JAVA
public int findMaxForm(String[] strs, int m, int n) {
        int[][] dp = new int[m + 1][n + 1];
        int[] num0 = new int[strs.length];
        int[] num1 = new int[strs.length];
        for(int i = 0; i < strs.length; i++){
            for (char c : strs[i].toCharArray()) {
                if (c == '0')
                    num0[i]++;
                else if(c == '1')
                    num1[i]++;
            }
        }
        //跟01背包不同，这里没有初始化！！而且如果初始化了，那样等于多计算一次就会错误！！
        for(int a = 0; a < strs.length; a++)
            for(int i = m; i >= num0[a]; i--){
                for(int j = n; j >= num1[a]; j--){
                    dp[i][j] =  Math.max(dp[i][j], dp[i - num0[a]][j - num1[a]] + 1);
                }
            }
        return dp[m][n];
    }
```

### LeetCode139. [单词拆分](https://leetcode-cn.com/problems/word-break/)

```JAVA
//递归法，但是超时，也许思路有用？
public boolean wordBreak(String s, List<String> wordDict) {
        if(s == null || s.isEmpty())
            return true;

        boolean result = false;
        for(int j = 0; j < wordDict.size(); j++){
            result = result || (prefix(s, wordDict.get(j)) && wordBreak(s.substring(wordDict.get(j).length()) , wordDict));
        }
        return result;

    }
    public boolean prefix(String str1, String str2){
        if(str1.length() < str2.length())
            return false;
        for(int i = 0, j = 0; i < str2.length(); i++, j++){
            if(str1.charAt(i) != str2.charAt(j))
                return false;
        }
        return true;
    }

//非递归版本，已AC
    public boolean wordBreak(String s, List<String> wordDict) {
        //s = "catsandog", wordDict = ["cats", "dog", "sand", "and", "cat"]
        //dp[i]表示子字符串(0,i)是否能够分割
        boolean dp[] = new boolean[s.length() + 1];
        //用于迭代
        dp[0] = true;
        //遍历每个长度的字符串
        for(int i = 1; i < dp.length; i++){
            //在字典中寻找是否能匹配
            for(int j = 0; j < wordDict.size(); j++){
                dp[i] = dp[i] || (postfix(s.substring(0, i),wordDict.get(j)) && dp[i - wordDict.get(j).length()]);
            }
        }
        return dp[s.length()];

    }
    //字符串str1和字符串str2从后面开始能否完全匹配
    //str1应该长于等于str2
    public boolean postfix(String str1, String str2){
        if(str1.length() < str2.length())
            return false;
        for(int i = str1.length() - 1, j = str2.length() - 1; j >= 0; i--, j--){
            if(str1.charAt(i) != str2.charAt(j))
                return false;
        }
        return true;
    }
```

### Leetcode494. [目标和](https://leetcode-cn.com/problems/target-sum/)

这个跟01背包类似，却不一样！

因为目标和可能来自比它大的，也可能来自比它小的

因此是通过某个元素去得到它能到达的值，而不是通过能到达它相邻某个元素的情况数得到该元素的值

**也就是通过原因推结果，而不是结果推原因！**

而且考虑到动态过程中的值未知性，不能通过数组存储，直接通过`HashMap`存储！

```java
     public int findTargetSumWays(int[] nums, int S) {
        HashMap<Integer, Integer> hashMap = new HashMap<>();
        if(nums.length < 1)
            return 0;
        //用第一个元素初始化迭代 HashMap，表示能到达0 + 这个元素，也能到达0 - 这个元素
        hashMap.put(-nums[0], hashMap.getOrDefault(-nums[0], 0) + 1);
        hashMap.put(nums[0],hashMap.getOrDefault(nums[0], 0) + 1);
        for(int i = 1; i < nums.length; i++){
            //新建临时的HashMap，防止修改过程中对此次迭代产生影响
            HashMap<Integer, Integer> newHashMap = new HashMap<>();
            //不是遍历数组，而是所有键的集合
            for(int j : hashMap.keySet()){
                //可以加这个元素
                newHashMap.put(j - nums[i],
                        newHashMap.getOrDefault(j - nums[i], 0) + hashMap.get(j)
                );
                //也可以减这个元素
                newHashMap.put(j + nums[i],
                        newHashMap.getOrDefault(j + nums[i], 0) + hashMap.get(j)
                );
            }
            hashMap = newHashMap;
        }
        return hashMap.getOrDefault(S, 0);
    }
```



### LeetCode84. [柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

**单调栈问题**

<https://zhuanlan.zhihu.com/p/26465701>

```JAVA
public int largestRectangleArea(int[] heights) {
    //维护一个单调递增的序列
    //遇到比尾元素小的元素就弹出一定的元素，再把它放进去
    Stack<Integer> stack = new Stack<>();
    int area = 0;
    for(int i = 0; i <= heights.length; i++){
        while(!stack.isEmpty() &&(i == heights.length || heights[stack.peek()] > heights[i])){
            //去掉最大的一个元素
            int h = heights[stack.pop()];
            int width = i;
            if(!stack.isEmpty())
                width = i - stack.peek() - 1;
            area = Math.max(h * width, area);
        }
        if(i < heights.length)
            stack.push(i);
    }
    return area;
}
```

