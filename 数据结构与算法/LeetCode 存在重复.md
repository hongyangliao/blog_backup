### 题目
给定一个整数数组，判断是否存在重复元素。

如果任何值在数组中出现至少两次，函数返回 true。如果数组中每个元素都不相同，则返回 false。

示例 1:

输入: [1,2,3,1]
输出: true
示例 2:

输入: [1,2,3,4]
输出: false
示例 3:

输入: [1,1,1,3,3,4,3,2,4,2]
输出: true

### 解答
```
class Solution {
    public boolean containsDuplicate(int[] nums) {
        int length = nums.length;
        // 当数组长度为0和1时不可能重复
        if(length == 0 || length == 1) {
            return false;
        }
        
       Set<Integer> set = new HashSet<Integer>();
        for(int x = 0; x <= length - 1; x++) {
            if(set.contains(nums[x])) {
                return true;
            } else {
                set.add(nums[x]);
            }
        }
        
       // for(int x = 0; x < length - 1; x++) {
       //     for(int y = x + 1; y <= length - 1; y++) {
       //         if(nums[x] == nums[y]) {
       //             return true;
       //         }
       //     }
       // }
        
        // quickSort(nums, 0, length - 1);
        
        // for(int x = 1; x < length; x++) {
        //     if(nums[x - 1] == nums[x]) {
        //         return true;
        //     }
        // }
        return false;
    }
    
    // 快排
    public void quickSort(int[] nums, int start, int end) {
        if(start < end) {
            int i = start;
            int j = end;
            int tmp = nums[i];
            while(i < j) {
                while(j > i && nums[j] >= tmp) j--;
                // 交换
                nums[i] = nums[j];
                while(i < j && nums[i] <= tmp) i++;
                // 交换
                nums[j] = nums[i];
            }
            nums[i] = tmp;
            // 递归调用左边
            quickSort(nums, start, i - 1);
            // 递归调用右边
            quickSort(nums, i + 1, end);
        }
        
    }
}
```

### 思路
这题拿到手最简单的想法就是
1.双重for循环遍历比较，时间复杂度为O(n^2)
2.另外一种为使用hash表来存储， 判断hash表中是否有重复的值
3.先排序，因为排序过后是有序数组，然后在比较相邻两个数是否相等，这里用的是快速排序，遗憾的是这个方法提交失败，说超时，快排也不一定快啊，如果已经是有序数组，这个时间复杂度最差为O(n^2),另外快排的平均时间复杂度为
O(nlogn)
4.使用Arrays.sort()方法先排序，跟3差不多，我觉得出题者意思肯定不是让你用api的
