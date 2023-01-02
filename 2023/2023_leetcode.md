# 2022-01-02


## 螺旋矩阵 II

* [59. 螺旋矩阵 II](https://leetcode.cn/problems/spiral-matrix-ii)

c++版本：

```
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> mat(n, vector<int>(n,0));
        int total = n*n;
        int num =1;
        int l=0, r=n-1, t=0, b=n-1;
        while(num<=total){
            for(int i=l;i<=r;i++){mat[t][i] = num++;}
            t++;
            for(int i=t;i<=b;i++){mat[i][r] = num++;}
            r--;
            for(int i=r;i>=l;i--){mat[b][i]= num++;}
            b--;
            for(int i=b;i>=t;i--){mat[i][l]= num++;}
            l++;
        }
        return mat;
    }
};
```

## 长度最小的子数组

* [209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/description/)

c++版本：

```c++
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int result = INT32_MAX;
        int leftIndex = 0;
        int sum = 0;
        int subLength = 0;

        for(int rightIndex = 0; rightIndex < nums.size(); rightIndex++){
            sum += nums[rightIndex];
            while(sum >= target){
                subLength = rightIndex - leftIndex +1;
                result = result < subLength ? result : subLength;
                sum -= nums[leftIndex++];
            }
        }
        return result == INT32_MAX ? 0 : result;
    }
};
```


## 有序数组的平方
* [977. 有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/description/)

c++版本：

```c++
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        vector<int> new_nums(nums.size(), 0);
        int lastIndex = nums.size()-1;
        
        int leftIndex = 0;
        int rightIndex = nums.size()-1;
        while(leftIndex <= rightIndex){
            int leftVal = nums[leftIndex] * nums[leftIndex];
            int rightVal = nums[rightIndex] * nums[rightIndex];
            if(leftVal < rightVal){
                new_nums[lastIndex--] = rightVal;
                rightIndex--;
            }
            else{
                new_nums[lastIndex--] = leftVal;
                leftIndex++;
            }
        }
        return new_nums;
    }
};
```

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