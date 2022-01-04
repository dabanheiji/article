[toc]

# leetCode刷题笔记

开始刷`leetCode`了啦，在这里记录一下每天的收获。

## 两数之和

解题思路：

1. 用和来减去数组每一项的值，并把结果与当前项的下标分别作为键值储存起来。
2. 以当前项为键去从对象中取值，看能不能取到下标，如果能取到，则结果为取到的下标与当前项的下标。

```js
[1, 3, 5, 8, 9, 11]
11

/**
 * 数组中每一项对应的下标分别是
 * 项  下标
 * 1 => 0
 * 3 => 1
 * 5 => 2
 * 8 => 3
 * 9 => 4
 * 11 => 5
 * 
 * 我们把和的值减去每一项的结果作为键
 * 每一项的下标作为值储存在对象中就会得到下面这样一个对象
 * map = {
 *  10: 0，
 *  8: 1,
 *  6: 2,
 *  3: 3,
 *  2: 4,
 *  0: 5
 * }
 * 
 * 我们在遍历的时候可以以当前项的值为键 看对象中是否存在这个键
 * 若存在则通过这个键获取到的值与当前项的下标就是答案
 * 以1为键 map.hasOwnProperty(1) === false 无值继续遍历
 * 以3为键 map.hasOwnProperty(3) === true 当前下标为1 map[3]为3 则答案为[1,3] 或 [3,1]
*/
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
const twoSum = (nums, target) => {
  let map = {};
  for(let i = 0, len = nums.length; i < len; i++) {
    const complement = target - nums[i];
    if(map.hasOwnProperty(nums[i])) {
      return [map[nums[i]], i];
    } else {
      map[complement] = i;
    }
  }
}
```

## 两数相加

这道题使用了链表这种数据类型，解题思路如下：

对两个链表遍历，让两个链表的对应项相加，结果如果大于10，则需进1，结果与10取余余数为当前项的值，我们需要将进的这个1储存起来，在下一位计算时求和的同时还需加上进的这一位。

```js
/**
 * l1: 4 -> 5 -> 3
 * l2: 5 -> 7 -> 6
 * 
 * 354 + 675 = 1029
 * 最终应输出 9 -> 2 -> 0 -> 1
 * 
 * 遍历两个链表
 * 第一次
 * l1  4
 * l2  5
 * 和  9 小于10 则当前值为9
 * 进位（和/10）向下取整 0
 * 
 * 第二次
 * l1  5
 * l2  7
 * 和  12 大于10 取余为 2 则当前值为 2
 * 进位 （和/10） 向下取整为 1
 * 
 * 第三次
 * l1 3
 * l2 6
 * 和 9 需要记得上次进了1，还需加上上次进位 为10，除以10区域 为 0，则当前值为 0
 * 进位 （和/10）向下取整 为 1
 * 
 * 遍历结束，这进位值为1 将1放置链表最后
 * 
 * 结束
*/
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var addTwoNumbers = (l1, l2) => {
  let list = new ListNode();
  let current = list, carry = 0;

  while (l1 !== null || l2 !== null) {
    let sum = 0;

    if(l1) {
      sum += l1.val;
      l1 = l1.next;
    }

    if(l2) {
      sum += l2.val;
      l2 = l2.next;
    }

    sum += carry;
    let val = sum % 10;
    current.next = new ListNode(val);
    current = current.next;
    carry = Math.floor(sum / 10);
  }

  if(carry) {
    current.next = new ListNode(carry)
  }

  return list.next;
}
```

## 无重复最长子子符串

查找一个字符串中最长无重复的子字符串，思路：

使用双指针，一个set集合，遍历字符串，如果集合中不存在该字符，将该字符存入set，使最大长度加一，如果set中存在该字符则已经开始重复，移除第二个指针当前对应元素后使第二个指针加1，然后继续判断，如此反复直至将字符串遍历完毕。

```js
/**
 * 假设字符串为  "abcabcbb"
 * 
 * 使用 i j两个指针，起始都指向字符串的第一个字符
 * 创建一个set集合
 * 
 * 遍历
 * 第一次
 * i -> "a"
 * j -> "a"
 * set {} 不存在 i 将 i对应字符 a 放入set 使 i 加1 更新最大长度 1 继续判断
 * 
 * 第二次
 * i -> "b"
 * j -> "a"
 * set {"a"} 中不存在 i 将i对应字符 b 放入 set 使 i 加1 更新最大长度 2 继续
 * 
 * 第三次
 * i -> "c"
 * j -> "a"
 * set {"a", "b"} 中不存在 i 将i对应字符 c 放入 set 使 i 加1 更新最大长度 3 继续
 * 
 * 第四次
 * i -> "a"
 * j -> "a"
 * set {"a", "b", "c"} 存在 i 
 * 则这个时候已经存在重复字符，将j对应元素删除，set {"b", "c"}
 * set中此时不存在 i 将 i 存入set {"b", "c", "a"}
 * 
 * 第五次
 * i -> "b"
 * j -> "b"
 * set {"b", "c", "a"} 存在 i 
 * 则这个时候已经存在重复字符，将j对应元素删除，set {"c", "a"} j加1
 * set中此时不存在 i 将 i 存入set {"c", "a", "b"}
 * 
 * 第六次
 * i -> "c"
 * j -> "c"
 * set {"c", "a", "b"} 存在 i 
 * 则这个时候已经存在重复字符，将j对应元素删除，set {"a", "b"} j加1
 * set中此时不存在 i 将 i 存入set {"a", "b", "c"}
 * 
 * 第七次
 * i -> "b"
 * j -> "b"
 * set {"a", "b", "c"} 存在 i 
 * 则这个时候已经存在重复字符，将j对应元素删除，set {"b", "c"} j加1
 * 这个时候set中仍存在 i 继续删除 set {"c"} j加1
 * set中此时不存在 i 将 i 存入set {"c", "b"}
 * 
 * 第八次
 * i -> "b"
 * j -> "c"
 * set {"a", "b", "c"} 存在 i 
 * 则这个时候已经存在重复字符，将j对应元素删除，set {"c"} j加1
 * 这个时候set中仍存在 i 继续删除 set {} j加1
 * set中此时不存在 i 将 i 存入set {"c"}
 * 
 * 返回最大长度 3
*/
var lengthOfLongestSubstring = s => {
  let len = s.length;

  if (len === 0) {
    return 0;
  }

  let set = new Set();
  let i = 0, j = 0, maxLength = 1;

  for(i; i < len; i++) {
    if (!set.has(s[i])) {
      set.add(s[i]);
      maxLength = Math.max(maxLength, set.size);
    } else {
      while (set.has(s[i])) {
        set.delete(s[j]);
        j++;
      }
      set.add(s[i])
    }
  }

  return maxLength
}
```

## 最长回文子字符串

解题思路：

从中间开始向前后两个方向开始扩散，判断前后邻位是否相同，如果相同则判断邻位的邻位直至不同或超出边界位置，记录前邻位下标以及最大长度，截取字符串。需要考虑回文字符串长度为单双数的差异。

```js
/**
 * 假设字符串为 "babad"
 * 
 * 以 b 为中心扩散，则左边界出界
 * 
 * 以 b，a 作为扩散邻位 b !== a 不符合回文字符串
 * 
 * 以 a 为中心 扩散 左邻 b 右邻 b 相等 左邻下标为 0， 回文字符串长度为3
 * 
 * 以 a，b 为扩散邻位 a !== b 不符题意
 * 
 * 以 b 为中心 扩散 左邻 a 右邻 a 左邻下标为 1 ，回文字符串长度为3
 * 
 * 以 b a 为扩散邻位 不符题意
 * 
 * 以 a 为中心扩散 左右邻位不等 不符题意
 * 
 * 以 a，d 为扩散邻位 不符题意
 * 
 * 以 d 为扩散中心 超出右边界
 * 
 * 结束截取 回文字符串 "bab"
 * 
*/
var longestPalindrome = s => {
  let len = s.length;
  if (len < 2) {
    return s;
  }

  let start = 0, maxLength = 1;

  function helper(left, right){
    while (left >= 0 && right < len && s[left] === s[right]) {
      let nowLen = right - left + 1;
      if (nowLen > maxLength) {
        maxLength = nowLen;
        start = left;
      }
      left--;
      right++;
    }
  }

  for (let i = 0; i < len; i++) {
    helper(i, i + 1); // 回文字符串长度为双数
    helper(i - 1, i + 1); // 回文字符串长度为单数
  }

  return s.substr(start, maxLength);
}
```

## 三数之和

解题思路：

这道题思路比较难想

1. 对数组进行排序，以便我们操作比较
2. 我们需要两个指针，在对每一项进行遍历的时候一个指针指向当前项的下一项，另一个指针指向数组的最后一项，当前项与两个指针对应的数字进行相加，如果和等于0，符合题意放入答案数组，第一个指针向后移，第二个指针向前移并需要进行重复判断，直至两个指针相同结束开始遍历下一项。

```js
/**
* [-1,0,1,2,-1,-4]
1. 排序
    [-4, -1, -1, 0, 1, 2]

    遍历第一项 两个指针为start  end
        当前项 -4
        start -1
        end 2
        三者相加 为 -3 小于0 start加一

        start -1 等于前一项跳过
        start 0 三者相加为 -2 start 加一
        start 1 三者相加 -1 start 加一等于end 跳过
    
    遍历第二项
        当前项 -1
        start -1
        end 2
        三者相加为0 将答案 [-1,-1,2]存入答案数组 start加1 end减一
        start 0
        end 1
        三者相加为0 将答案 [-1,0,1] 存入答案 start加一 end减一 start > end 跳过

    遍历第三项
        当前项 -1 等于上一项跳过
    
    遍历第四项
        重复上述步骤
*/
var threeSum = (nums) => {
  let result = [];

  nums.sort((a,b) => a - b);

  for(let i = 0; i < nums.length - 2; i++) {
    let start = i + 1, end = nums.length - 1;
    if(i === 0 || nums[i] !== nums[i - 1]){
      while (start < end) {
        let sum = nums[i] + nums[start] + nums[end];
        if (sum === 0) {
          result.push([nums[i], nums[start], nums[end]]);
          start++;
          end--;
          while (start < end && nums[start] === nums[start - 1]) {
            start++;
          }
          while (start < end && nums[end] === nums[end + 1]) {
            end--;
          }
        } else if (sum > 0) {
          end--;
        } else if (sum < 0) {
          start++;
        }
      }
    }
  }

  return result;
}
```

## 删除链表倒数第n个节点

解题思路：

这又是一道链表的题目，这道题有一个十分巧妙的解法，我们可以使用双指针，第一个指针指向第一个节点，第二个指针指向第n + 1个节点，然后开始遍历，当第二个指针指向最后一个节点的时候，第一个指针刚好指向倒数第n + 1个节点，这时候让第一个指针的next往后移两次跳过倒数第n个节点即可。

```js
/**
1 -> 2 -> 3 -> 4 -> 5
n = 2
删除倒数第二项即删除4

指针 i  j
i 指向 1
j 指向 n + 1项  3

遍历
i 2
j 4

i 3
j 5 为最后一项此时 i 为删除项的前一项

i.next = i.next.next 即可
*/
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
var removeNthFromEnd = (head, n) => {
  const list = new ListNode();
  list.next = head;

  let n1 = list, n2 = list;

  // 重置 n2 位置
  for (let i = 0; i < n; i++) {
    n2 = n2.next;
  }

  while(n2.next !== null){
    n1 = n1.next;
    n2 = n2.next;
  }

  n1.next = n1.next.next;

  return list.next;
}
```

## 有效的括号

解题思路：

这道题目可以利用栈的特性，将左括号作为键，右括号作为值储存简历对应关系，然后遍历字符串如果是左括号就把对应的右括号存入栈中，此时剩余字符串中下一个右括号必须与取出栈的最后一个括号相等，否则就是不符题意，如果遍历完毕栈中仍存在括号，那么就代表有括号没闭合，不符题意如果遍历完毕栈是空的则符合题意

```js
/**
  "{([])}"

  // 建立映射关系
  {
    "{": "}",
    "[": "]",
    "(": ")",
  }

  遍历
  第一项
    在键中存在，将值存入栈
    栈 ["}"]
  
  第二项
    在键中存在，将值存入栈
    栈 ["}", ")"]
  
  第三项
    在键中存在，将值存入栈
    栈 ["}", ")", "]"]
  
  第四项
    键中不存在，因为栈中最后一项是最后出现的左括号的配对项，所以从栈中取出与之比较 相等
    栈 ["}", ")"]
  
  第五项
    键中不存在，取出最后一项比较 相等
    栈 ["}"]
  
  第六项
    键中不存在 取出最后一项比较 相等
    栈 []
  
  栈长度为0则代表所有括号都有配对，符合题意
*/
function isValid(s) {
  // 建立映射关系
  const map = new Map();
  map.set('(', ')')
  map.set('[', ']')
  map.set('{', '}')

  // 将数组当成我们的栈
  let stack = [];

  for (let i = 0, len = s.length; i < len; i++) {
    if (map.has(s[i])) {
      stack.push(map.get(s[i]));
    } else {
      if (stack.pop() !== s[i]) {
        return false
      }
    }
  }

  if (stack.length) {
    return false
  }

  return true
}
```

## 合并两个有序链表

解题思路：

将两个链表的每一项比较排序，直至一个链表结束，然后将另一个链表剩余部分放入排好序的链表中。

```js
/*
1 -> 2 -> 4
1 -> 3 -> 5 -> 6

比较
1 等于 1 前两个顺序无所谓
1 -> 1

2 小于 3 将 3 放入 2后
1 -> 1 -> 2 -> 3

4小于5 将5 放入 4 后
1 -> 1 -> 2 -> 3 -> 4 -> 5

链表1已经结束，将链表2剩余部分放入排序链表之后
1 -> 1 -> 2 -> 3 -> 4 -> 5 -> 6
*/
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
function mergeTwoLists(l1, l2) {
  let dummy = new ListNode();
  let curr = dummy;

  while (l1.val !== null && l2.val !== null) {
    if (l1.val > l2.val) {
      curr.next = l2;
      l2 = l2.next;
    } else {
      curr.next = l1;
      l1 = l1.next;
    }
    curr = curr.next;
  }

  if (l1.val) {
    curr.next = l1;
  }

  if (l2.val) {
    curr.next = l2;
  }

  return dummy.next;
}
```

## 两两交换链表节点

解题思路：

0 -> 1 -> 2 -> 3

假设我们交换1和2个节点的位置，那么我们需要做下面步骤

1. 让0 指向 2
2. 让1 指向 2 的后一个节点 即 3
3. 让2 指向 1
4. 将指针移至1

然后我们一直重复这个步骤即可

```js
/*
1 -> 2 -> 3 -> 4

我们互换两个相邻节点的位置，那么我们需要获取这两个节点的前一个节点， 所以我们需要先创建一个0节点
0 -> 1 -> 2 -> 3 -> 4

1. 让0 指向 2
2. 让1 指向 2 的后一个节点 即 3
3. 让2 指向 1
4. 将指针移至1

0 -> 2 -> 1 -> 3 -> 4

1. 让1 指向 4
2. 让4 指向 3 的后一个节点 即 null
3. 让4 指向 3
4. 将指针移至3

0 -> 2 -> 1 -> 4 -> 3
*/
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
function swapPairs(head) {
  const dummy = new ListNode();
  dummy.next = head;
  let current = dummy;

  while (current.next !== null && current.next.next !== null) {
    let n1 = current.next;
    let n2 = current.next.next;
    current.next = n2;
    n1.next = n2.next;
    n2.next = n1;
    current = n1;
  }

  return dummy.next;
}
```

## 字母异位词分组

解题思路：

这道题的解题思路是让使用字母和每个字母相同的单词能生成一个相同的key，然后将这些单词放入数组中作为value建立映射关系，最后依次放入结果数组中去。

```js
/*
  1. 对入参单词数组进行遍历，对每一个单词都生成一个长度为26的数组，每个数组对应该字母出现的次数
  2. 将数组转换为字符串作为key， 设置一个数组放置当前字符串作为value，后续再出现同样的key代表是字母异位词，就将当前单词添加进value中
  3.将建立的映射关系中所有value添加至结果数组中返回
*/
function groupAnagrams(strs) {
  if(strs.length === 0) {
    return [];
  }

  const map = new Map();

  for (const str of strs) {
    const keyArr = Array(26).fill(0);
    for (let i = 0, len = str.length; i < len; i++) {
      const asi = str.charCodeAt(i) - 97; // a的ASCII码为97，减去97后正好对应数组的下标
      keyArr[asi] += 1;
    }
    const key = keyArr.join('-');
    if (map.has(key)) {
      map.set(key, [...map.get(key), str])
    } else {
      map.set(key, [str])
    }
  }

  const result = [];

  for (const [_, v] of map) {
    result.push(v)
  }

  return result;
}
```

## 最大子数组和

解题思路：

```js
/*
[-2,1,-3,4,-1,2,1,-5,4]

将每一项都映射成到当前位置最大的和，放入另一个数组
var maxSum = []

第一项
-2
maxSum [-2]

第二项
1
与前一项在和数组中的数相加，与当前值比较 -2 + 1 < 1，到当前为止最大和为1，后续都重复这个比较
maxSum [-2, 1]

第三项
-3 
-3 + 1  = -2 > -3
maxSum [-2, 1, -2]

第四项
4
4 + -2 = 2 < 4
第四项位置最大和为4
maxSum [-2, 1, -2, 4]

第五项
-1
4 + -1 = 3 > -1
maxSum [-2, 1, -2, 4, 3]

第六项
2
3 + 2 = 5 > 2
maxSum [-2, 1, -2, 4, 3, 5]

第七项
1
5 + 1 = 6 > 1
maxSum [-2, 1, -2, 4, 3, 5, 6]

第八项
-5
6 + -5 = 1 > -5
maxSum [-2, 1, -2, 4, 3, 5, 6, 1]

第九项
4
1 + 4 = 5 > 4
maxSum [-2, 1, -2, 4, 3, 5, 6, 1, 4]

取出maxSum中最大数 6
*/
function maxSubArray(nums) {
  let memo = [], max = nums[0];
  memo[0] = nums[0];

  for(let i = 1, len = nums.length; i < len; i++) {
    memo[i] = Math.max(nums[i] + memo[i - 1], nums[i]);
  }

  max = Math.max(...memo);

  return max;
}
```

## 螺旋矩阵

解题思路：

这道题目比较朴实无华，按照螺旋顺序走下去依次将每一项放入结果数组即可。

```js
/*
[1, 2, 3],
[4, 5, 6],
[7, 8, 9],

顺序是 右 下 左 上
走完右 需要将上边界加1
走完下 需要将右边界减1
走完左 需要将下边界减1
走完上 需要将左边界加1
依次遍历
*/
function spiralOrder(matrix) {
  if (matrix.length === 0) {
    return [];
  }

  let left = 0, 
      top = 0, 
      right = matrix[0].length - 1, 
      bottom = matrix.length - 1;
  
  let direction = 'right', result = [];

  while (left <= right && top <= bottom) {
    if (direction === 'right') {
      for (let i = left; i <= right; i++) {
        result.push(matrix[top][i])
      }
      top++;
      direction = 'down';
    } else if (direction === 'down') {
      for (let i = top; i <= bottom; i++) {
        result.push(matrix[i][right])
      }
      right--;
      direction = 'left'
    } else if (direction === 'left') {
      for (let i = right; i >= left; i--) {
        result.push(matrix[bottom][i])
      }
      bottom--;
      direction = 'up'
    } else if (direction === 'up') {
      for (let i = bottom; i >= top; i--) {
        result.push(matrix[i][left])
      }
      left++;
      direction = 'right'
    }
  }

  return result;
}
```

## 跳跃游戏

解题思路：

这道题的解题方法比较多，可以使用动态规划也可以使用贪心算法，而动态规划又可以分为`top-down`以及`bottom-up`两种写法

### 动态规划top-down解法

```js
/*
top-down;
我们需要一个另一个数组去记录原数组中每一项是否能达到终点
如果能 则用 1表示， 不能用-1表示， 未知用0表示
所以第一步是需要创建一个等长的数组并且全部填充为0
因为最后一项是终点，所以我们创建的数组最后一项我们也需要设置为1
top-down的写法我们需要将每一种情况都算进去，然后判断当前节点是否可行，需要用到递归
一个节点能走的路中，只要有一条路能走通，那么这个节点就是ok的，并且前面的节点只要能走到ok节点，那么这个节点就也是ok的
对每种情况进行递归，在递归中计算每一情况即可
*/
function canJump(nums) {
  const total = nums.length;
  const memo = Array(total).fill(0);
  memo[total - 1] = 1; // 最后一位是终点，是ok节点

  function jump(position) {
    // 如果当前节点是ok节点之间返回true，是bad节点返回false
    if (memo[position] === 1) {
      return true;
    } else if (memo[position] === -1) {
      return false
    }

    // 判断当前节点o不ok
    // 防止超过数组长度，出现越界问题
    const maxJump = Math.min(position + nums[position], total - 1);

    // 遍历当前节点能走的所有道路
    for (let i = position + 1; i <= maxJump; i++) {
      // 如果有一条路走的通，证明当前节点ok，返回true
      const result = jump(i);
      if (result) {
        memo[position] = 1;
        return true
      }
    }

    // 走到这里代表这个节点所有路都走不通，返回false
    memo[position] = -1;
    return false;
  }

  // 我们只需要看第0项能不能走到终点
  return jump(0);
}
```

### 动态规划bottom-up解法

```js
/*
bottom-up;
上面的top-down是从前往后去排查的，而bottom则是从后往前去走；
我们同样需要创建一个记录各个节点状态的数组，最后一位是ok的
然后如果倒数第二位也是ok的的话，那么我们只需要判断其他节点能不能到倒数第二位就可以了
*/
function canJump(nums) {
  const total = nums.length;
  const memo = Array(total).fill(0);
  memo[total - 1] = 1;

  // 因为最后一位必定是ok的，所以这里从倒数第二位开始
  for (let i = total - 2; i >= 0; i--) {
    // 最大的跳跃次数，防止越界
    const maxJump = Math.min(i + nums[i], total - 1);
    for (let j = i + 1; j <= maxJump; j++) {
      // 如果当前节点下有一条路走得通，就更新当前节点为ok
      if (memo[j] === 1) {
        memo[i] = 1;
        break;
      }
    }
  }

  return memo[0] === 1;
}
```

### 贪心算法解法

```js
/*
其实解这道题有一个很重要的思想，只要这个节点是ok的，那么能跳到这个节点的节点也是ok的，贪心算法就是我们从后往前找最靠前的能到终点的节点，如果这个节点是第一个，那么就返回true
*/
function canJump(nums) {
  const total = nums.length;
  let minCanJumpNode = total - 1;

  for (let i = total - 1; i >= 0; i--) {
    const maxJump = i + nums[i]; // 当前节点能跳跃的最大步数
    // 能跳跃的最大步数如果大于最小成功节点的下标则能代表当前节点也是成功节点，更新最小成功节点为当前下标
    if (maxJump >= minCanJumpNode) {
      minCanJumpNode = i;
    }
  }

  return minCanJumpNode === 0;
}
```

## 合并区间

解题思路：

将数组针对第一项进行排序，排序之后，记录当前合并的区间，如果后一项的第一项比当前合并的区间的后一项小，则说明可以合并，否则将当前合并的区间存入结果数组，重新创建合并区间

```js
function merge(intervals) {
  if (intervals.length < 2) {
    return intervals;
  }

  intervals.sort((a, b) => a[0] - b[0]);

  let result = [], curr = intervals[0];

  for(let interval of intervals) {
    if(interval[0] <= curr[1]) {
      curr[1] = Math.max(interval[1], curr[1]);
    } else {
      result.push(curr);
      curr = interval;
    }
  }

  if (curr.length > 0) {
    result.push(curr);
  }

  return result;
}
```

## 插入区间

解题思路：

将新区间push进原数组排序，然后执行上题步骤

```js
var insert = function(intervals, newInterval) {
  intervals.push(newInterval);
  intervals.sort((a, b) => a[0] - b[0]);

  let result = [], curr = intervals[0];

  for(let interval of intervals) {
    if (curr[1] >= interval[0]) {
      curr[1] = Math.max(curr[1], interval[1])
    } else {
      result.push(curr);
      curr = interval;
    }
  }

  if(curr.length) {
    result.push(curr);
  }

  return result;
}
```

## 不同路径

解题思路：

这道题我们可以将格子看作数组中的一个元素，然后我们可以将地图转换成一个二维数组，数组每个元素就是到达自身的路径条数，然后让我们计算某个位置的值。

```js
/*
m = 3, n = 7;
等价于
[
  [1, 1, 1, 1, 1, 1, 1],
  [1, x, x, x, x, x, x],
  [1, x, x, x, x, x, ?]
]

这道题我们可以看作求 ? 位置的数是多少的问题
首先我们可以确定的是第一条数组的每一项都是1
并且每个数组的第一项都是1
然后其他格子中，每个格子的数应该等于这个位置上面一个位置与左边一个位置的和
即：
[
  [1, 1, 1, 1,  1,  1,  1],
  [1, 2, 3, 4,  5,  6,  7],
  [1, 3, 6, 10, 15, 21, 28]
]
*/
function uniquePaths(m, n) {
  const memo = [];
  for(let i = 0; i < m; i++) {
    memo.push([])
  }

  for(let row = 0; row < m; row++) {
    memo[row][0] = 1;
  }

  for(let col = 0; col < n; col++) {
    memo[0][col] = 1;
  }

  for(let row = 1; row < m; row++) {
    for(let col = 1; col < n; col++) {
      memo[row][col] = memo[row - 1][col] + memo[row][col - 1];
    }
  }

  return memo[m - 1][n - 1];
}
```

## 加一

解题思路：

这道题十分简单，直接看代码即可

```js
function plusOne(digits) {
  let carry = 1;

  for(let i = digits.length - 1; i >= 0; i--) {
    let sum = digits[i] + carry;
    digits[i] = sum % 10;
    carry = Math.floor(sum / 10);
  }

  if(carry) {
    digits.unshift(carry);
  }

  return digits;
}

// 或者
var plusOne = function(digits) {
  for(let i = digits.length - 1; i >= 0; i--) {
    if(digits[i] === 9) {
      digits[i] = 0;
    } else {
      digits[i]++;
      return digits
    }
  }

  digits.unshift(1);
  return digits;
};
```

## 爬楼梯

解题思路：

这道题第一个台阶只有一种走法，第二个台阶只有两种走法，因为最多一次只能走两步，所以第三个台阶只能从第一个台阶走两步，或者从第二个台阶走一步，即符合斐波拉且，实则是在求斐波拉且的第n项

```js
function climbStairs(n) {
  let memo = [];
  memo[1] = 1;
  memo[2] = 2;
  for(let i = 3; i <= n; i++) {
    memo[i] = memo[i-1] + memo[i-2];
  }

  return memo[n];
}
```

## 矩阵置零

解题思路：

如果可以使用新的数组那么本题将会很简单，但是题目上限制了不能开辟新的数组，所以有些麻烦，本题解题思路非常的长需要记住以下步骤：

1. 判断矩阵第一行和第一列有没有0，分别用两个变量储存
2. 从第二行第二列开始对矩阵进行遍历，如果当前位置为0，则将当前行的第一列与当前列的第一行设置为0
3. 再次遍历矩阵，如果当前行第一列是0或者当前列第一行是0，则将此位置设置为0
4. 判断第一步储存的状态，如果第一行存在0将第一行全部设置为0，判断第一列是否为0，如果是将第一列全部设置为0

```js
function setZeroes(matrix) {
  let firstRowHasZero = false;
  let firstColHasZero = false;

  // 1.
  for(let i = 0; i < matrix.length; i++) {
    if(matrix[i][0] === 0) {
      firstColHasZero = true;
    }
  }

  for(let i = 0; i < matrix[0].length; i++) {
    if(matrix[0][i] === 0) {
      firstRowHasZero = true;
    }
  }

  // 2. 
  for(let row = 1; row < matrix.length; row++) {
    for(let col = 1; col < matrix[0].length; col++) {
      if(matrix[row][col] === 0) {
        matrix[row][0] = 0;
        matrix[0][col] = 0;
      }
    }
  }

  // 3. 
  for(let row = 1; row < matrix.length; row++) {
    for(let col = 1; col < matrix[0].length; col++) {
      if(matrix[row][0] === 0 || matrix[0][col] === 0) {
        matrix[row][col] = 0;
      }
    }
  }

  // 4. 
  if(firstRowHasZero) {
    for(let i = 0; i < matrix[0].length; i++) {
      matrix[0][i] = 0;
    }
  }

  if(firstColHasZero) {
    for(let i = 0; i < matrix.length; i++) {
      matrix[i][0] = 0;
    }
  }

  return matrix;
}
```

##  删除排序链表中的重复元素

解题思路：

这道题目是比较简单的，因为给我们的是一个有序的链表，所以处理起来非常方便，我们只需要对链表进行遍历，判断当前节点的值是否等于下一节点的值，如果相等就删除下一个节点直至遍历完毕即可

```js
function deleteDuplicates(head) {
  let curr = head;

  while(curr && curr.next) {
    if(curr.val === curr.next.val) {
      curr.next = curr.next.next;
    } else{
      curr = curr.next;
    }
  }

  return head;
}
```

## 反转链表

解题思路：

因为单向链表是不可逆的，所以我们如果直接将当前节点的指针修改向前一个节点，那么则会丢失下一个节点，所以这时候我们就需要一个新的指针去保存下一个节点。所以这里需要使用三个指针。

```js
/*
  prev curr next

  反转需要牢记下面四步
  1. next = curr.next;
  2. curr.next = prev;
  3. prev = curr;
  4. curr = next;
*/
function reverseList(head) {
  let prev = null;
  let curr = head, next;

  while(curr) {
    next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
  }

  return prev;
}
```

然而在JavaScript中我们可以使用解构赋值，通过解构赋值的方式可以帮助我们节省去next这个指针

```js
function reverseList(head) {
  let prev = null;
  let curr = head;

  while(curr) {
    [prev, curr.next, curr] = [curr, prev, curr.next]
  }

  return prev;
}
```

## 反转链表2

解题思路：

这道题和上道反转链表十分相似，但是多了一点，就是反转中间的部分链表之后如何将中间的这部分与两边的部分连接起来，所以这里就又在上题的基础上又需要多使用两个指针。

```js
/*
1 -> 2 -> 3 -> 4 -> 5
2,4

1 -> 4 -> 3 -> 2 -> 5

如果按照反转链表1的思路会得到

1  2 <- 3 <- 4  5

然后在连接 1 5的时候就会异常吃力
我们在按照反转链表1的方法反转后最后curr指针会在5的位置，prev会在4的位置
所以我们还需要知道1和2的位置，因此我们在反转前需要使用另外两个指针指向1和2

最后我们需要注意一种特殊情况，如果反转的起始位置是1的话那么就会head就会改变，这时候要记得修改head
*/
function reverseBetween(head, left, right) {
  let prev = null, curr = head, next = null;

  for(let i = 1; i < left; i++) {
    prev = curr;
    curr = curr.next;
  }

  const prev1 = prev, curr1 = curr;

  for(let i = left; i <= right; i++) {
    next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
  }

  if(prev1) {
    prev1.next = prev
  } else {
    head = prev
  }

  curr1.next = curr;

  return head
}
```

## 买卖股票的最佳时机

解题思路：

这道题让计算最大利润，那么获取最大利润的时机一定在某个波峰，而每个波峰的最大利润都是当前波峰减去左侧最低的波谷，也就是数组左侧的最小值，所以只需要计算每个位置距离自己左侧最低点的值取最大即可。

```js
function maxProfit(prices) {
  if(prices.length === 0) {
    return 0;
  }

  let maxProfit = 0, minPrice = prices[0];

  for(let i = 0; i < prices.length; i++) {
    minPrice = Math.min(minPrice, prices[i]);
    maxProfit = Math.max(maxProfit, prices[i] - minPrice);
  }

  return maxProfit;
}
```

## 买卖股票的最佳时机2

解题思路：

这道题针对上一题解除了只能进行一次交易的限制，那么最大利润其实非常容易想明白，就是所有的波谷到波峰的差距的总和，这道题有两种解法

```js
/*
  解法一： 计算每个波谷到波峰的差价相加
*/
function maxProfit(prices) {
  if(prices.length === 0) {
    return 0;
  }

  let profit = 0, min = prices[0], max = prices[0];

  let i = 0;

  while (i < prices.length - 1) {
    while(i < prices.length - 1 && prices[i] >= prices[i + 1]) {
      i++
    }
    min = prices[i];
    while(i < prices.length - 1 && prices[i] <= prices[i + 1]) {
      i++
    }
    max = prices[i];
    profit += max - min;
  }

  return profit;
}
```

解法2： 贪心

```js
function maxProfit(prices) {
  if(prices.length === 0) {
    return 0;
  }

  let profit = 0;

  for(let i = 0; i < prices.length - 1; i++) {
    if(prices[i] < prices[i + 1]) {
      profit += prices[i + 1] - prices[i];
    }
  }

  return profit;
}
```

## 整数反转

解题思路：

这道题目其实很简单，最暴力的解法是讲数字变成字符串，然后反转，这里笔者的思路如下：

1. 将这个数与10的余数取出来，将余数使用result变量储存
2. 将这个数减去1中余数并除以10
3. 继续取余并将result乘以10之后加上新余数，反复至这个数为0为止

```js
/*
x = 123

result = 0
1. 与10取余  余数为3  result = 3
2. 将x减去余数并除以10  (123 - 3) / 10 = 12
3. 继续取余 余数为2 result乘以10再加上余数 3 * 10 + 2 = 32
重复
*/
function reverse(x) {
  let result = 0;

  while(x !== 0) {
    result = (result * 10) + (x % 10);
    x -= x % 10;
    x /= 10;
  }

  if(result > Math.pow(2, 31) || result < Math.pow(-2, 31) - 1) {
    return 0;
  }

  return result;
}
```

## 回文数

解题思路：

这道题笔者的思路是将数字转换成字符串，然后去判断

```js
var isPalindrome = function(x) {
  if(x < 0) {
    return false;
  }

  x = String(x);
  let left = 0, right = x.length - 1;

  while(x[left] === x[right] && left < right) {
    left++;
    right--;
  }

  if(left === right || left === right + 1) {
    return true;
  }

  return false;
}

// 或更简单的

var isPalindrome = function(x) {
  return x.toString() === x.toString().split('').reverse().join('');
}
```

## 盛最多水的容器

解题思路：

这道题面积是由下标差与较小的高相乘得到的，所以我们可以移动较小的边去计算面积会不会增大

```js
function maxArea(height) {
  let maxArea = 0;
  let left = 0, right = height.length - 1;

  while(left < right) {
    maxArea = Math.max(
      maxArea,
      Math.min(height[left], height[right]) * (right - left)
    );

    if(height[left] < height[right]) {
      left++
    } else {
      right--
    }
  }

  return maxArea
}
```

## 整数转罗马数字

解题思路：
先把5，50，500这几个特殊的数用键值对存，然后其他数与对于符号分别放在两个数组里；

1. 开始遍历数组，要是**数字小于当数组前项**，获取到数字与当前项的商
2. 商的可能性为 1-9 （商为0在第一步中已经被过滤了），逐一分析
  - 商为1-3 的情况一样，把当前项对应的符号重复1-3遍即可
  - 商为4 把数组当前项乘以5为键获取map中对应的符号，然后左侧放上当前项对应的符号
  - 商为9 数组前一项对应的符号，左侧放上当前项对应的符号
  - 商为5 以数组当前项乘以5为键从map中查找对应符号
  - 商为6-8 把数组当前项乘以5为键获取map中对应的符号，然后右侧放上当前项对应的符号重复商-5遍
3. 每轮遍历结束前需把当前拼好的数字减去。

```js
var intToRoman = function(num) {
  const map = new Map();

  map.set(5, 'V');
  map.set(50, 'L');
  map.set(500, 'D');

  const nums = [1000, 100, 10, 1];
  const symbols = ['M', 'C', 'X', 'I'];

  let result = '';

  for(let i = 0; i < nums.length; i++) {
    if(num >= nums[i]) {
      let count = Math.floor(num / nums[i]);

      if(count < 4) {
        result += symbols[i].repeat(count);
      } else if(count === 4) {
        result += `${symbols[i]}${map.get(nums[i] * 5)}`;
      } else if(count === 5) {
        result += map.get(nums[i] * 5);
      } else if(count < 9) {
        result += `${map.get(nums[i] * 5)}${symbols[i].repeat(count - 5)}`;
      } else {
        result += `${symbols[i]}${symbols[i - 1]}`;
      }

      num -= nums[i] * count
    }
  }

  return result;
}
```

## 罗马数字转数字

解题思路：
先简历键值对关系，对字符串从右往左遍历，获取当前位对应的数字，与上一位的数字，要是当前位的数字小于上一位则表示这是9或4，就减去当前位，否则加当前位，口齿不麻利，大概是这个意思。

```js
var romanToInt = function(s) {
  const map = new Map();
  map.set('I', 1);
  map.set('V', 5);
  map.set('X', 10);
  map.set('L', 50);
  map.set('C', 100);
  map.set('D', 500);
  map.set('M', 1000);
  
  let prev = 0, res = 0;

  for(let i = s.length - 1; i >= 0; i--) {
    const num = map.get(s[i]);
    if(num < prev) {
      res -= num;
    } else {
      res += num;
    }
    prev = num;
  }

  return res;
}
```

## 最长公共前缀

解题思路：

暂时使用暴力解法，待有更好的办法后继续补充

```js
var longestCommonPrefix = function(strs) {
  if(strs.length < 1) {
    return '';
  }

  let prefix = '';

  let s = strs[0], i = 0;

  while(i < s.length) {
    for(let str of strs) {
      if(str[i] !== s[i]) {
        return prefix;
      }
    }

    prefix += s[i];
    i++;
  }

  return prefix;
}
```

## 电话号码的字母组合

解题思路：

组合就是上一个组合的每一项分别与下一字母对应的所有项拼接而成

即`['a']` 与 `['b', 'c']`则需要把b，c分别加在a后结果为`['ab', 'ac']`，
想明白这一点就比较好解决了，我们让结果数组默认为第一个数字对应的字母然后使每个字母都拼接商下一项对应的字母们即可。

```js
var letterCombinations = function(digits) {
  if(digits.length === 0) {
    return [];
  }

  const map = new Map();
  map.set('2', ['a', 'b', 'c']);
  map.set('3', ['d', 'e', 'f']);
  map.set('4', ['g', 'h', 'i']);
  map.set('5', ['j', 'k', 'l']);
  map.set('6', ['m', 'n', 'o']);
  map.set('7', ['p', 'q', 'r', 's']);
  map.set('8', ['t', 'u', 'v']);
  map.set('9', ['w', 'x', 'y', 'z']);

  let result = map.get(digits[0]);

  for(let i = 1; i < digits.length; i++) {
    result = result.map(item => {
      return map.get(digits[i]).map(s => `${item}${s}`)
    }).flat(2);
  }

  return result;
}
```

## 验证回文串

解题思路：

这道题思路很简单，从第一位与最后一位，使用双指针遍历比较即可，唯一的难点在于转换字符串

```js
var isPalindrome = function(s) {
  s = s.toLocaleLowerCase().replace(/[\W_]/g, '');

  let left = 0, right = s.length - 1;

  while(left < right) {
    if(s[left] === s[right]) {
      left++;
      right--;
    } else {
      return false;
    }
  }

  return true
}
```