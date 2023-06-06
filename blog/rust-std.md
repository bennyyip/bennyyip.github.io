---
title: "Rust 标准库阅读笔记"
published_date: "2018-03-09 10:03:47 +0800"
layout: post.liquid
is_draft: true
data:
  route: blog
---
#
不定期更新……

---

# [std::colloections::HashMap](https://doc.rust-lang.org/std/collections/struct.HashMap.html)
读代码之前先看文档，其对`HashMap`的介绍如下：
> A hash map implemented with linear probing and Robin Hood bucket stealing.
>
> By default, HashMap uses a hashing algorithm selected to provide resistance against HashDoS attacks. The algorithm is randomly seeded, and a reasonable best-effort is made to generate this seed from a high quality, secure source of randomness provided by the host without blocking the program. Because of this, the randomness of the seed depends on the output quality of the system's random number generator when the seed is created. In particular, seeds generated when the system's entropy pool is abnormally low such as during system boot may be of a lower quality.
>
>The default hashing algorithm is currently SipHash 1-3, though this is subject to change at any point in the future. While its performance is very competitive for medium sized keys, other hashing algorithms will outperform it for small keys such as integers as well as large keys such as long strings, though those algorithms will typically not protect against attacks such as HashDoS.


同时文档提供了提供了三篇文章 / 论文，对理解代码很有帮助：
1. Pedro Celis. ["Robin Hood Hashing"](https://cs.uwaterloo.ca/research/tr/1986/CS-86-14.pdf)
2. Emmanuel Goossaert. [ "Robin Hood hashing" ](http://codecapsule.com/2013/11/11/robin-hood-hashing/)
3. Emmanuel Goossaert. [ "Robin Hood hashing: backward shift deletion" ](http://codecapsule.com/2013/11/17/robin-hood-hashing-backward-shift-deletion/)

## tricks
Naive 的实现可以用 `Vec<Option<u64, K, V>>` 来实现，显然这是缓存不友好的，因为每个元素都会有一个`Option`的 tag。这里标准库做了优化，`t.hashes[i] == 0` 表示该 bucket 为空。而 hash 函数计算出来的 hash 会通过`SafeHash`类型将其变成一个非 0 的值，即将最高位置为 1。


## 插入

## 删除

## 查找

## Resize/Reserve
```rust
/// Reserves capacity for at least `additional` more elements to be inserted
/// in the `HashMap`. The collection may reserve more space to avoid
/// frequent reallocations.
///
/// # Panics
///
/// Panics if the new allocation size overflows [`usize`].
///
/// [`usize`]: ../../std/primitive.usize.html
///
/// # Examples
///
/// ```
/// use std::collections::HashMap;
/// let mut map: HashMap<&str, isize> = HashMap::new();
/// map.reserve(10);
/// ```
#[stable(feature = "rust1", since = "1.0.0")]
pub fn reserve(&mut self, additional: usize) {
    let remaining = self.capacity() - self.len(); // this can't overflow
    if remaining < additional {
        let min_cap = self.len().checked_add(additional).expect("reserve overflow");
        let raw_cap = self.resize_policy.raw_capacity(min_cap);
        self.resize(raw_cap);
    } else if self.table.tag() && remaining <= self.len() {
        // Probe sequence is too long and table is half full,
        // resize early to reduce probing length.
        let new_capacity = self.table.capacity() * 2;
        self.resize(new_capacity);
    }
}
```
`self.table.tag()` 的值在每次插新 bucket 的时候更新，如果 `DIB` 超过 `const DISPLACEMENT_THRESHOLD: usize = 128;` 则将其改为真，维持持一个低的 `DIB`。

```
/// Resizes the internal vectors to a new capacity. It's your
/// responsibility to:
///   1) Ensure `new_raw_cap` is enough for all the elements, accounting
///      for the load factor.
///   2) Ensure `new_raw_cap` is a power of two or zero.
#[inline(never)]
#[cold]
fn resize(&mut self, new_raw_cap: usize) {
    assert!(self.table.size() <= new_raw_cap);
    assert!(new_raw_cap.is_power_of_two() || new_raw_cap == 0);

    let mut old_table = replace(&mut self.table, RawTable::new(new_raw_cap));
    let old_size = old_table.size();

    if old_table.size() == 0 {
        return;
    }

    let mut bucket = Bucket::head_bucket(&mut old_table);

    // This is how the buckets might be laid out in memory:
    // ($ marks an initialized bucket)
    //  ________________
    // |$$$_$$$$$$_$$$$$|
    //
    // But we've skipped the entire initial cluster of buckets
    // and will continue iteration in this order:
    //  ________________
    //     |$$$$$$_$$$$$
    //                  ^ wrap around once end is reached
    //  ________________
    //  $$$_____________|
    //    ^ exit once table.size == 0
    loop {
        bucket = match bucket.peek() {
            Full(bucket) => {
                let h = bucket.hash();
                let (b, k, v) = bucket.take();
                self.insert_hashed_ordered(h, k, v);
                if b.table().size() == 0 {
                    break;
                }
                b.into_bucket()
            }
            Empty(b) => b.into_bucket(),
        };
        bucket.next();
    }

    assert_eq!(self.table.size(), old_size);
}
```
