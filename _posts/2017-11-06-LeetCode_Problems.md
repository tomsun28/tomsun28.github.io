---
layout: post
title: LeetCode_Problems
data: 2017-11-06
tag: algorithm
---

# 在LeetCode上做过的题  

<br>
### 1. two sum  

![](/images/posts/leetcode/image1.PNG)  
```
class Solution {
    public int[] twoSum(int[] nums, int target) {
        
        for(int i =0;i<nums.length;i++){
            for(int j=i+1;j<nums.length;j++){
                if(nums[i]+nums[j]==target){
                    
                    return new int[]{i,j};
                }
            }            
        }
        return null;
        
    }
}

```
<br>

### 2. add two numbers  

![](/images/posts/leetcode/image2.PNG)  
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode l3 = new ListNode(0);
        ListNode p = l3;
        int up = 0;
        while(l1!=null||l2!=null||up!=0)
        {
            int x = (l1!=null)? l1.val : 0;
            int y = (l2!=null)? l2.val : 0;
            
            p.next = new ListNode((x+y+up)%10);
            up = (x+y+up)/10;
            p = p.next;
            if(l1!=null) 
            {
                l1 = l1.next;
            }
            if(l2!=null) 
            {
                l2 = l2.next;
            }
        }
        
        return l3.next;
    }
}

```
<br>

### 3. longest substring without repeating characters  

![](/images/posts/leetcode/image3.PNG)  
```
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int maxSub = 0;
        List<Character> list = new ArrayList<>();
        for(int i=0;i<s.length();i++){
            int j =i;
            int sum=0;
            while(j<s.length()&&!list.contains(s.charAt(j))){
                sum++;
                list.add(s.charAt(j++));
                
            }
            maxSub = maxSub<sum?sum:maxSub;
            list.clear();
        }
        return maxSub;
    }
}
 
```
<br>

### 4. median of two sorted arrays  

![](/images/posts/leetcode/image4.PNG)  
```
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int[] nums3 = new int[nums1.length+nums2.length];
        int i=0;
        for(int x:nums1){
            nums3[i]=x;
            i++;
        }
        for(int x:nums2){
            nums3[i]=x;
            i++;
        }
        Arrays.sort(nums3);
        if(i==0)return i/1.0;
        if(i==1)return nums3[0]/1.0;
        if(i%2==0)
            return (nums3[i/2]+nums3[i/2-1])/2.0;
        else return nums3[(i-1)/2]/1.0;
    }
}

```

```
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int m = nums1.length;
        int n = nums2.length;
        int[] mn = new int[m+n];
        int i=0,j=0,z=0;
        while(i<m&&j<n)
        {
            if(nums1[i]<=nums2[j])
            {
                mn[z++]=nums1[i++];
            }
            else
            {
                mn[z++]=nums2[j++];
            }
        }
        while(i<m)
        {
            mn[z++]=nums1[i++];
                
        }
        while(j<n)
        {
            mn[z++]=nums2[j++];
        }
        if((m+n)%2==1)
            return mn[(m+n+1)/2-1];
        else
            return (mn[(m+n)/2]+mn[(m+n)/2-1])/2.0;
    }
}
```

<br>

### 5. longest palindromic substring  

![](/images/posts/leetcode/image5.PNG)  
```
class Solution {
    public String longestPalindrome(String s) {
        int start=0,end=0;
        for(int i=0;i<s.length();i++)
        {
            int len1 = compareIndexExpore(s,i,i);
            int len2 = compareIndexExpore(s,i,i+1);
            int len = Math.max(len1,len2);
            if(len>end-start){
                start = i-(len-1)/2;
                end = i+len/2;
    
            }
            
        }
        return s.substring(start,end+1);
        
    }
    
    public int compareIndexExpore(String s,int i,int j){
        
        while(i>=0&&j<s.length()&&s.charAt(i)==s.charAt(j)){
            i--;
            j++;
        }
        return j-i-1;
    }
}

```
<br>

### 6. zigZag conversion  

![](/images/posts/leetcode/image6.PNG)  
```
class Solution {
    public String convert(String s, int numRows) {
        if(numRows<=1)
        {
            return s;
        }
        
        StringBuffer[] buffer = new StringBuffer[numRows];
        for(int m=0;m<buffer.length;m++)
            buffer[m]=new StringBuffer();
        int index = 0;
        int flag = 0;
        int change = 0;
        for(int i = 0;i<s.length();i++)
        {
            buffer[index].append(s.charAt(i));
            if(flag==0)
            {
                if(index==numRows-1)
                {
                    flag=1;
                    change=-1;
                }
                else
                {
                    change=1;
                }
            }
            if(flag==1)
            {
                if(index==0)
                {
                    flag=0;
                    change=1;
                }
                else
                {
                    change=-1;
                }
                
            }
                index+=change;   
        }
        StringBuffer combine = new StringBuffer();
        for(StringBuffer temp:buffer)
        {
            combine.append(temp.toString());
        }
        return combine.toString();
    }
}


```
<br>

### 7. reverse integer  

![](/images/posts/leetcode/image7.PNG)  
```
class Solution {
    
    private static int sum;
    public int reverse(int x) {
        int result=0,newresult=0;
        int tail=0;
        while(x!=0)
        {
            tail = x%10;
            newresult = result*10+tail;
            if((newresult-tail)/10!=result)
                return 0;
            result = newresult;
            x=x/10;
        }
        return result;
    }
}


```
<br>

### 8. string to integer(atoi)  

![](/images/posts/leetcode/image8.PNG)  
```
class Solution {
    public int myAtoi(String str) {
        int index=0,sign=1,total=0;
        //1
        if(str.length()==0)
            return 0;
        
        //2
        while(str.charAt(index)==' '&&index<str.length())
            index++;
        
        //3
        if(str.charAt(index)=='+'||str.charAt(index)=='-')   
        {
            sign = str.charAt(index)== '+' ? 1 : -1;
            index++;
        }
        //4
        while(index<str.length())
        {
            int digit = str.charAt(index) - '0';
            if(digit<0||digit>9)
                break;
            if(Integer.MAX_VALUE/10<total||Integer.MAX_VALUE/10==total&&Integer.MAX_VALUE%10<digit)
               {
                   return sign == 1 ? Integer.MAX_VALUE : Integer.MIN_VALUE;
               } 
            total = total*10 + digit;
            index++;
        
        }
        return total*sign;
}
}


```
<br>

### 9. palindrome number  

![](/images/posts/leetcode/image9.PNG)  
```
class Solution {
    public boolean isPalindrome(int x) {
        int sum = 0;
        int x_temp = x;
        int digit = 0;
        while(x > 0)
        {
            digit = x%10;
            sum = sum*10 + digit;
            x = x/10;
        }
        return sum==x_temp ? true:false;
    }
}


```
<br>

### 10. regular expression matching  

![](/images/posts/leetcode/image10.PNG)  
```



```
<br>

### 11. container with most water  

![](/images/posts/leetcode/image11.PNG)  
```



```
<br>

### 12. integer to roman  

![](/images/posts/leetcode/image12.PNG)  
```



```
<br>

### 13. roman to integer  

![](/images/posts/leetcode/image13.PNG)  
```



```
<br>

### 14. longest common prefix  

![](/images/posts/leetcode/image14.PNG)  
```



```
<br>

### 15. 3sum  

![](/images/posts/leetcode/image15.PNG)  
```



```
<br>

### 16. 3sum closest  

![](/images/posts/leetcode/image16.PNG)  
```



```
<br>

### 17.two sum  

![](/images/posts/leetcode/image17.PNG)  
```



```
<br>





