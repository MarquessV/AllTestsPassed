---
layout: post
title: "LeetCode #146: LRU Cache"
date: 2017-11-22 10:45:00
use_math: true
tags: [cpp, data-structures, hard, leetcode]
---

[Source: LeetCode](https://leetcode.com/problems/lru-cache/description/)

Design and implement a data structure for Least Recently Used (LRU) cache. It should support the following operations: get and put.

`get(key)` - Get the value (will always be positive) of the key if the key exists in the cache, otherwise return -1.

`put(key, value)` - Set or insert the value if the key is not already present. When the cache reached its capacity, it should invalidate the least recently used item before inserting a new item.

**Follow up:**

Could you do both operations in \\(\mathcal{O}(1)\\) time complexity?

**Example**:

<div class="example">
LRUCache cache = new LRUCache( 2 /* capacity */ );<br/>
cache.put(1, 1);<br/>
cache.put(2, 2);<br/>
cache.get(1);       // returns 1<br/>
cache.put(3, 3);    // evicts key 2<br/>
cache.get(2);       // returns -1 (not found)<br/>
cache.put(4, 4);    // evicts key 1<br/>
cache.get(1);       // returns -1 (not found)<br/>
cache.get(3);       // returns 3<br/>
cache.get(4);       // returns 4
</div>

## Introduction

This is one of my favorite problems on LeetCode because it rewards designing before implementing. Hacking away at this problem is only going to cause headaches. If you haven't figured it out yet, I highly recommend you keep trying. Here are some hints:

1. Choose the right data structures. This is the key. Think about what data you are being given and what the operations you need to perform on them are. What structures fit the bill? Good choices here make your life way easier further on.

2. Once you choose your structures, start designing the methods with pseudocode. Is there any repetition that you could refactor into a separate method?

3. Use custom testcases to add method calls one at a time to see where your implementation goes wrong.

## Approach

First, we should start thinking about what data structures to use to represent our LRU Cache. To guide our thinking we need to fully understand the data we are working with and the operations we will perform on that data.

### Choosing the Right Structures

First off, our LRU Cache has a capacity which is provided to the constructor. So we'll need to store that value as an integer.

In addition, the LRU Cache holds key-value pairs. This is a hint that some type of associative array would be of use here. The first associative array I always look at is a hash map since it offers an expected constant runtime for `search`, `insert`, and `delete` operations. Now we have to look ahead and consider the potential limitations of using a hash table. The worst case runtime is $\mathcal{O}(n)$ for all operations and this will happen a lot if the hash function is bad or the table has to be resized often. Fortunately, if we use a STL implementation of a hash table we are guaranteed to have a good hash function and our cache will always have a fixed size so we don't have to worry about resizing if we make it large enough from the start. Another limitation of hash tables is that its contents aren't ordered. This *is* a problem since we need to maintain some kind of order regarding the least recently used object. We don't want to give up the speed of a hash table so this limitation drives our choice in what structure to use in conjunction with the hash table.

Our next choice needs to fill in the gaps left by the hash table. With a hash table we can quickly search for a value given any key but we can't store the objects in order of least recently used. Clearly, some type of ordered list is necessary. The list needs to be able to quickly move an object that was recently used to the top of the list. Arrays `insert` and `delete` in linear time. This certainly isn't ideal since, barring a special case, we need to move an object every time we use our LRU cache. This completely ruins any advantage we had by using a hash table. A linked list allows us to `insert` and `delete` in constant time and are an appealing choice. Again, we consider the limitations. In this case, we should be concerned with the linear access times in a list. Well, the hash map gives us constant access times to the value of a given key. What if we mapped the key to the node in the linked list containing the value for that key. We'd be able to find the node and move it in constant time!

At this point, we're ready to define the members of our data structure. I've decided to define my own doubly-linked list. Pointers to both the head and tail of the list allow us to access both the most recently and least recently used objects quickly. I used the C++ STL [`unordered_map`](http://en.cppreference.com/w/cpp/container/unordered_map) as my hash table and `int` to hold the capacity.

Note: Since capacity would never be negative, `std::size_type` would be the best choice for `maxCapacity` in a serious implementation. However, LeetCode gives capacity as an int by default so we roll with that.

{% highlight cpp %}
struct node {
  int key;
  int value;
  node* prev;
  node* next;
  node(int k, int v, node* p, node* n) {
    key = k;
    value = v;
    prev = p;
    next = n;
  }
};

node* head; // most recently used
node* tail; // least recently used

unordered_map<int, node*> pairs;

int maxCapacity;

{% endhighlight %}

## Writing the Methods

### Constructor

  Our constructor should always initialize an instance of the object to a usable state. To do this, we need to set `maxCapacity` to the given capacity. In addition, we need to set the `head` and `tail` pointers to `nullptr` because the compiler wont do that for us. Finally, we use the `reserve` method on `pairs` to create enough buckets in our hash table to avoid having to resize the table at any other point in the instances lifetime.

{% highlight cpp %}
LRUCache(int capacity) {
  maxCapacity = capacity;
  head = tail = nullptr;
  pairs.reserve(capacity);
}
{% endhighlight %}

### Get

This one should be straightforward enough. We see if `key` exists in `pairs`. If it does we move the node with the value to the head of the list and return the value. Otherwise, we just return -1. At some point in this process you might have realized that making a node the head of the list is going to be an operation repeated multiple times in this implementation. We need to do it in this method, and we will need to do it whenever we add a new key-value pair to the cache. We should plan to make a separate method for this to avoid writing the same code twice.

{% highlight cpp %}
int get(int key) {
  auto res = pairs.find(key);
  if(res == pairs.end()) {
    return -1;
  }
  makeHead(res->second);
  return res->second->value;
}
{% endhighlight %}

Abstracting the `makeHead` method and leaving it for later will make this method and `put` easier to write and allows us to avoid writing the same code twice.

If any of this syntax is a mystery to you it probably has to do with iterators. If you have no clue about iterators, consider reading [this](https://www.cprogramming.com/tutorial/stl/iterators.html) great tutorial on iterators and [this](https://www.cprogramming.com/c++11/what-is-c++0x.html) tutorial about the `auto` keyword in C++11. Once iterators start to make sense, you can read about how the `find` method works for `unordered_map`s specifically [here](http://en.cppreference.com/w/cpp/container/unordered_map/find).

### Put

This method has a few cases to consider. 

**Case 1:** The key already exists in the cache. In this case we need to update the value, and make the corresponding node the head of the list.

**Case 2:** If capacity hasn't been reached, then we simply make a new node with the value, make it the head of the list, and place it in `pairs`.

**Case 3:** Otherwise, the capacity has been reached and we need to get rid of the least recently used object in the cache. The LRU object is pointed to by `tail`. So we need to remove the corresponding entry in `pairs`, update `tail`, and delete the object.

{% highlight cpp %}
void put(int key, int value) {
  auto res = pairs.find(key);
  if(res != pairs.end()) { // Case 1
    res->second->value = value;
    makeHead(res->second);
  } else if(pairs.size() < maxCapacity) { // Case 2
      node* toInsert = new node(key, value, head, nullptr);
      makeHead(toInsert);
      pairs.emplace(key, toInsert);
  } else { // Case 3
      pairs.erase(tail->key)
      node* toRemove = tail;
      tail = tail->prev;
      if(pairs.size() != 0) {
        tail->next = nullptr;
      }
      delete toRemove;
      put(key, value);
  }
}
{% endhighlight %}

We use the size of pairs to see if we hit capacity or not. In addition, the capacity could be 1 which is why we need to check if `pairs.size() != 0` before trying to null out the next pointer of tail. If we didn't perform this check we would end up trying to access a null pointer.

Notice the recursion at the end of case 3. By removing `key` from `pairs` we've made sure that `key` isn't in pairs and the size of `pairs` will be less than `maxCapacity`. Thus, this call will always end up in case 2, which appropriately adds the key-value pair to the cache.

### makeHead

Finally, we have to implement our `makeHead` method. This requires some tricky pointer manipulation but as long as we are careful it isn't a hard method to implement.

Again, we consider the possible cases.

**Case 1:** `n` is the head. Nothing needs to be done.

**Case 2:** There is only 1 item in `pairs`. The one item must be `n` and we make `n` both the `head` and the `tail`.

**Case 3:** `n` isn't the head, and there is more than 1 item in the list. Then there are 3 subcases:

* **Case 3a:** `n` is the tail node. In this case we need to update the tail to be `n`'s predecessor. The new tail then needs to point to `nullptr` rather than `n`.

* **Case 3b:** `n` is not the tail and it is not a new node. We need to connect `n`'s predecessor and successor before moving it (otherwise we would break the list into two pieces).

* **Case 3c:** `n` is a new node. `n` isn't in the list yet, so we don't have to worry about maintaining the continuity of the list, just inserting it at the head.

After any of these subcases, we make the current `head`'s predecessor `n` and set `n`'s successor to be the current head. Finally, we null out `n`'s predecessor and make `n` the head.

{% highlight cpp %}
void makeHead(node* n) {
  if(n == head) {
    return;
  }
  if(!head || data.size() == 1) {
    head = n;
    tail = n;
    return;
  }
  if(n == tail) {
    tail = tail->prev;
    tail->next = nullptr;
  } else if(n->next) {
    n->next->prev = n->prev;
    n->prev->next = n->next;
  }
  head->prev = n;
  n->next = head;
  n->prev = nullptr;
  head = n;
}
{% endhighlight %}

## Complexity Analysis

### Constructor

This is the most costly part of our implementation since `pairs.reserve(capacity)` has a worstcase of $\mathcal{O}($`capacity`$^2)$. However doing this once here avoids doing it multiple times when we are putting items in the cache. 

Going forward, let `n = capacity`.

### makeHead

No loops or recursion here. The only function call is when we check the size of `pairs` which is guaranteed $\mathcal{O}(1)$. The rest of the function are conditional statements and asssignments so we have a guaranteed constant runtime.

### get

Finding a key in `pairs` is expected to run in $\mathcal{O}(1)$ time and $\mathcal{O}(n)$ in the worst case. However, the worst case is extremely unlikely since having a good hash function and appropriately sized table should cause negligible collisions. Other than that, there is one call to `makeHead`, a conditional statement and two return statements. All of which are $\mathcal{O}(1)$. So we expect $\mathcal{O}(1)$ but if we are *very very* unlucky the call could run in $\mathcal{O}(n)$.

### put

Everything here is $\mathcal{O}(1)$, except for finding, inserting, and erasing from `pairs` which has an expected constant runtime but a linear worst-case runtime. However, we still expect $\mathcal{O}(1)$ for the same reasons regarding `find` stated above.

### Overall

In general, a user could expect every call to the LRUCache to run in $\mathcal{O}(1)$ time. While $\mathcal{O}(n)$ is possible it is extremely unlikely. The probability of getting the same value using the [`std::hash`](http://en.cppreference.com/w/cpp/utility/hash) function is very small. This fact in conjunction with using `reserve` to create enough buckets in `pairs` for our cache we expect to see effectively zero collisions.

As far as the results in LeetCode go, this implementation ran in 82ms and beat out 89% of other submissions.

## Conclusion

We've created a functional, fast, and hopefully easy to understand LRU Cache. All methods perform the expected task, it is essentially an $\mathcal{O}(1)$ implementation, and we've appropriately named and structured our code in a meaningful and logical way.
