title: "Rust 标准库阅读笔记"
published_date: "2018-03-9 10:03:47 +0800"
layout: post.liquid
is_draft: true
data:
  route: blog
---
# 
不定期更新……

---

# [std::colloections::HashMap](https://doc.rust-lang.org/std/collections/struct.HashMap.html)
> A hash map implemented with linear probing and Robin Hood bucket stealing.   
>
> By default, HashMap uses a hashing algorithm selected to provide resistance against HashDoS attacks. The algorithm is randomly seeded, and a reasonable best-effort is made to generate this seed from a high quality, secure source of randomness provided by the host without blocking the program. Because of this, the randomness of the seed depends on the output quality of the system's random number generator when the seed is created. In particular, seeds generated when the system's entropy pool is abnormally low such as during system boot may be of a lower quality.  
> 
>The default hashing algorithm is currently SipHash 1-3, though this is subject to change at any point in the future. While its performance is very competitive for medium sized keys, other hashing algorithms will outperform it for small keys such as integers as well as large keys such as long strings, though those algorithms will typically not protect against attacks such as HashDoS.


提供了三篇文章：
1. Pedro Celis. ["Robin Hood Hashing"](https://cs.uwaterloo.ca/research/tr/1986/CS-86-14.pdf)
2. Emmanuel Goossaert. [ "Robin Hood hashing" ](http://codecapsule.com/2013/11/11/robin-hood-hashing/)
3. Emmanuel Goossaert. [ "Robin Hood hashing: backward shift deletion" ](http://codecapsule.com/2013/11/17/robin-hood-hashing-backward-shift-deletion/)

