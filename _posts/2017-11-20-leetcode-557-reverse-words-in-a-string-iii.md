---
layout: post
title: "LeetCode #557: Reverse Words in a String III"
date: 2017-11-20 11:30:00
use_math: true
tags: [LeetCode, Strings, C++]
---

[Source: LeetCode](https://leetcode.com/problems/reverse-words-in-a-string-iii/description/)

Given a string, you need to reverse the order of characters in each word within a sentence while still preserving whitespace and initial word order.

Example:

>**Input**: "Let's take LeetCode contest"
>
>**Output**: "s'teL ekat edoCteeL tsetnoc"

Note: In the string, each word is separated by single space and there will not be any extra space in the string.

## Approach

The basic idea is to:

1. Delimit the string by spaces
2. For every word: append the reversed word to the output string.

## Simple STL Solution

This algorithm is slightly slower than the second solution here, but it takes advantage of the C++ standard library. The idea behind this is that an easy to write and easy to understand solution can sometimes be better than a slightly faster hard to write and understand solution in the real world. In addition, by keeping the STL in mind, we can avoid "reinventing the wheel."

The first important piece of this algorithm is the [`istringstream`](http://en.cppreference.com/w/cpp/io/basic_istringstream/basic_istringstream) function from `<sstream>`. `istringstream` takes a string as an input and converts it to an input stream. Since input streams only feed in characters up to the next whitespace we can easily process one word at a time.

Secondly, we use the [`reverse`](http://en.cppreference.com/w/cpp/algorithm/reverse) function from `<alogrithm>` to reverse the characters in each word. `reverse` takes two iterators as parameters and will reverse the items between the iterators.

### Code

{% highlight cpp %}
string reverseWords(string s) {
  istringstream words(s);
  string word;
  words >> word;
  s = "";
  while(words) {
    reverse(word.begin(), word.end());
    s += word;
    words >> word;
    if(words) {
      s += " ";
    }
  }
  return s;
}
{% endhighlight %}

To start, we convert the string into an istringstream `words`. Then we read in the first word into `word`.

Next, we overwrite `s` to the empty string so that we can concatenate the output string onto it. By overwriting `s` we save space and since `s` is passed by value and not by reference we don't lose any information by doing this.

Notice we use the input stream `words` as the sentinel to the while loop. An istream returns `nullptr` if either the failbit or badbit is set. These bits represent the stream being in an error state. For the purposes of this problem, the only error occurs when we read after the last word `words`. This means that the while loop will terminate after we read the last word.
If you are curious you can read more about how `istringstream`s resolve to boolean values [here](http://en.cppreference.com/w/cpp/io/basic_ios/operator_bool). 

Inside the while loop, we reverse `word` and concatenate it to `s`. Next, we read in the next word and if it didn't take us past the end of the input stream, then we add a space. If we didn't perform this check we would concatenate an extra space onto `s`.

### Runtime

Regardless of how many words are in the string, we end up reversing every character that isn't a space. Since `reverse` has a runtime of $\mathcal{O}(n)$ our algorithm also has the same runtime where $n$ is the length of `s`. Since there is no way to avoid reversing the string, and the fastest known reversal algorithm is $\mathcal{O}(n)$ we can't expect any sort of asymptotic improvement.

This submission ran in 23ms on LeetCode and beat out about 48% of C++ submissions. This is about average, but we can do better.

## Faster solution

While there isn't an asymptotic improvement over $\mathcal{O}(n)$ we can still optimize the algorithm to run a little faster. 

The main idea is that we don't need to waste anytime delimiting the string. We can keep track of the beginning of each word with an iterator. When we find a space, we reverse the string from the beginning of the word to the space, and point the beginning iterator to one after the space. We repeat this until we've processed the entire string.

### Code

{% highlight cpp %}
string reverseWords(string s) {
  string::iterator start = s.begin(), current = s.begin(), end = s.end();
  while(current != end) {
    current++;
    if(*current == ' ' || current == end) {
      reverse(start, current);
      start = current+1;
    }
  }
  return s;
}
{% endhighlight %}

We use a couple of iterators:

`start` will point to the beginning of the most recent word we are processing.

`current` points to the current character in the string we are looking at. It may seem like a good idea to start at `s.begin()+1`, but by starting at the beginning we can elegantly handle the case where `s == ""`.

`end` just holds the iterator `s.end()`. This is to avoid the overhead of using the getter method repeatedly. This overhead is either minimal or non-existent but every millisecond we shave off in this challenge dramatically increases its ranking so we include it.

The while loop continues if `current` isn't pointing to `s.end()`, exactly one after the end of `s`. Since `end` is exactly one after the end of the string we check if `current != end` and we increment `current` at the beginning of the loop rather than at the end. If we used `start != end` or incremented `current` at the end of the loop we'd end up going past `end` and thus never exit the while loop.

### Runtime

This submission ran at a blazing 19ms! This a 17% improvement which realistically doesn't matter. Still, it is faster than almost 88% of C++ submissions.
