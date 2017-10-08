extends: post.liquid

title:   关于异或运算的一些 tricks
date:    19 Aug 2016 23:02:52 +0800
route:   blog
---

# 异或运算的一些性质
- 交换律: `a ^ b = b ^ a`
- 结合律: `a ^ (b ^ c) = b ^ c`
- 异或是异或的逆运算: `a ^ b = c <=> b ^ c = a`

推论:
- `x ^ 0 = x`
- `x ^ x = 0`

阅读下文之前请务必把上述几个性质搞明白.
下面介绍几个比较 tricky 的应用, 越往下越复杂.

---

# 简单的加密
由于异或的逆运算是它本身, 因此可以实现简单的加密. (其实有逆运算的运算就可以用来加密)  
设明文 `M`, 密钥 `S`, 密文 `C`
加密: `C = M ^ S `
解密: `M = C ^ S`  

# 交换两个数  

```cpp
a = a ^ b;
b = a ^ b; // a ^ b ^ b == a
a = a ^ b; // a ^ b ^ a == b
```
这是一个广为人知却不实用的技巧, 比用中间变量的方法慢且不通用.

# Single Number
> Given an array of integers, every element appears twice except for one. Find that single one.

> Note:
> Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

这是 leetcode 上的[一道题目](https://leetcode.com/problems/single-number/), 利用异或的性质可以得到一个相当简洁的解.

```cpp
class Solution {
 public:
  int singleNumber(vector<int>& nums) {
    int result = 0;
    for (int i : nums) {
      result^=i;
    }
    return result;
  }
};
```
# 实现双向链表
> **CLRS 10.2-8**
> Explain how to implement doubly linked lists using only one pointer value x.np per item instead of the usual two (next and prev). Assume that all pointer values can be interpreted as k-bit integers, and define x.np = x.next XOR x.prev, the k-bit "exclusive-or" of x.next and x.prev. (The value NIL is represented by 0). Be sure to describe what information you need to access the head of the list. Show how to implement the SEARCH, INSERT and DELETE operations on such a list. Also show how to reverse such a list in O(1) time.

之所以写这篇 blog 其实就是因为刚刚写完这道题, 觉得很新奇. 这是一种特殊的双向链表, 一般的双向链表的结点是有 `prev` 和 `next` 两个指针的, 而它只有一个 `np` 指针, `np = prev ^ next`. 由于 `xor` 是 `xor` 的逆运算, 所以 `np` `prev` `next` 三者可以知二求一, 即:  
`np = prev ^ next`
`prev = np ^ next`
`next = prev ^ np`   
这样实现的链表空间复杂度是寻常链表的一半. 而在时间复杂度方面, 插入, 查找, 删除都是 `O(n)` 的, 除了在头尾时是 `O(1)`. 但是它的 `reverse` 操作是 `O(1)` 的, 只需把 `head` 和 `tail` 交换. 下面它是 C++ 的实现.

```cpp
#include <iostream>

// a ^ b = c ==> b ^ c = a
struct node {
  int key;
  node* np;  // np = prev xor next
  node(int val, node* _np = nullptr) : key(val), np(_np) {}
};

class xor_list {
 public:
  xor_list() : head(nullptr), tail(nullptr) {}

  void insert(int val) {
    if (!head) {
      auto to_insert = new node(val);
      head = to_insert;
      tail = to_insert;
      return;
    }
    if (tail) {
      auto to_insert = new node(val, ptr_xor(tail, nullptr));
      tail->np = ptr_xor(prev_ptr(tail, nullptr), to_insert);
      tail = to_insert;
      return;
    }
  }

  node* search(int val) {
    node* prev = nullptr;
    node* next;
    auto ptr = head;
    while (ptr) {
      next = next_ptr(prev, ptr);
      if (ptr->key == val) return ptr;
      prev = ptr;
      ptr = next;
    }
    return nullptr;
  }

  node* remove(node* to_remove) {
    node* prev = nullptr;
    node* next;
    auto ptr = head;
    while (ptr) {
      next = next_ptr(prev, ptr);
      if (ptr == to_remove) break;
      prev = ptr;
      ptr = next;
    }
    if (ptr == nullptr) return nullptr;
    prev->np = ptr_xor(prev_ptr(prev, ptr), next);
    next->np = ptr_xor(prev, next_ptr(ptr, next));
    delete ptr;
    return next;
  }

  void reverse() {
    auto tmp = head;
    head = tail;
    tail = tmp;
  }

  void show() {
    node* prev = nullptr;
    auto ptr = head;
    while (ptr) {
      std::cout << ptr->key << ", ";
      auto next = ptr_xor(prev, ptr->np);
      prev = ptr;
      ptr = next;
    }
    std::cout << std::endl;
  }

 private:
  node* ptr_xor(node* lhs, node* rhs) {
    return (node*)((unsigned long)lhs ^ (unsigned long)rhs);
  }

  node* prev_ptr(node* curr, node* next) { return ptr_xor(curr->np, next); }
  node* next_ptr(node* prev, node* curr) { return ptr_xor(prev, curr->np); }

 private:
  node* head;
  node* tail;
};

// test
int main() {
  xor_list xl;
  for (int i = 0; i < 10; ++i) {
    xl.insert(i);
  }
  xl.show();
  xl.remove(xl.search(4));
  xl.show();
  xl.reverse();
  xl.show();
}
```
