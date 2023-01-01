# 2022-01-01

## 移除元素

* [27. 移除元素](https://leetcode.cn/problems/remove-element/description/)

c++版本：
```c++
// 快慢指针法

class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int slowIndex =0;
        for(int fastIndex =0; fastIndex < nums.size(); fastIndex++){
            if(val != nums[fastIndex]){
                nums[slowIndex++]=nums[fastIndex];
            }
        }
        return slowIndex;
    }
};
```


```c++
//相向双指针法

class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int leftIndex =0;
        int rightIndex = nums.size()-1;

        while(leftIndex <= rightIndex){
            while(leftIndex <= rightIndex && val != nums[leftIndex]){
                leftIndex++;
            }

            while(leftIndex <= rightIndex && val == nums[rightIndex]){
                rightIndex--;
            }

            if(leftIndex < rightIndex){
                nums[leftIndex++] = nums[rightIndex--];
            }
        }
        
        return leftIndex;
    }
};
```


## 二分查找_(35. 搜索插入位置)
* [35. 搜索插入位置](https://leetcode.cn/problems/search-insert-position/description)

c++版本：
```c++
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size()-1;

        while(left <= right){
            int mid = left + (right-left)/2;
            if(nums[mid] > target){
                right = mid -1;
            }
            else if(nums[mid] < target){
                left = mid + 1;
            }
            else{
                return mid;
            }
        }

        return left;
    }
};
```

## 二分查找_(34. 在排序数组中查找元素的第一个和最后一个位置)
* [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array)

java版本：
```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int result[]={-1,-1};
        int left = 0;
        int right = nums.length-1;

        while(left <= right){
            int mid = left + (right-left)/2;
            if(nums[mid] > target){
                right = mid-1;
            }
            else if(nums[mid] < target){
                left = mid+1;
            }
            else{
                int start=mid;
                int end =mid;
                while(start >0 && nums[start-1]==target){
                    start--;
                }
                while(end<nums.length-1 && nums[end+1]==target){
                    end++;
                }
                result[0]=start;
                result[1]=end;
                break;
            }            
        }
        return result;
    }
}
```