### 描述
>请实现有重复数字的升序数组的二分查找
给定一个 元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的第一个出现的target，如果目标值存在返回下标，否则返回 -1

### 示例

    输入：
    [1,2,4,4,5],4
    返回值：2 
    说明：从左到右，查找到第1个为4的，下标为2，返回2

### 代码
```java
import java.util.*;

public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * 如果目标值存在返回下标，否则返回 -1
     * @param nums int整型一维数组 
     * @param target int整型 
     * @return int整型
     */
    public int search (int[] nums, int target) {
        // write code here
        int low = 0;
        int high = nums.length-1;
        int mid = 0;
        while(low <= high) {
            mid = (low + high)/2;
            if(nums[mid] == target) {
                while(mid != 0 && nums[mid-1] == nums[mid]) {
                    mid--;
                }
                return mid;
            }
            else if(nums[mid] > target) {
                high = mid - 1;
            } else {
                low = mid + 1 ;
            }
        }
        return -1;
        
    }
}
```