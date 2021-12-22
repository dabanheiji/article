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
