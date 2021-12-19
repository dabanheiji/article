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