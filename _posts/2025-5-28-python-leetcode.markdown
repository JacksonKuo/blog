---
layout: post
title: "Python: Leetcode Practice II"
date: 2025-5-28
tags: ["python"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Solve some Leetcode problems using Python: [https://leetcode.com/problemset/](https://leetcode.com/problemset/)

# Leetcode
Private Repo: [https://github.com/JacksonKuo/scripts-leetcode](https://github.com/JacksonKuo/scripts-leetcode)

* [https://leetcode.com/problems/palindrome-partitioning/](https://leetcode.com/problems/palindrome-partitioning/)
* [https://leetcode.com/problems/count-vowel-substrings-of-a-string/](https://leetcode.com/problems/count-vowel-substrings-of-a-string/)

#### Palindrome Substring - Fixed
```python
import re

#s = 'abcdcxy' write a function that returns true, if s contains a substring that is a palindrome of length equal to or greater than k

#the previous scripts only check if the length was exactly k. not equal to or greater

def clean(str):
    lower = str.lower()
    return re.sub("[^a-z]","",lower)

def isPalindrome(input):
    cleaned = clean(input)
    #print(cleaned)
    return check(cleaned)

def check(str):
    length = int(len(str)/2)
    for s in range(length):
        #print("s: ", s)
        first = str[s]
        second = str[len(str)-s-1]
        if first != second:
            return False
    else:
        return True

def get_substrings(s, k):
    last_index = len(s) - k
    total_loops = last_index + 1
    #print(last_index)
    for c in range(total_loops):
        substring = s[c:]
        substring_len = len(substring)
        #print(substring)
        sub_loops = substring_len - k + 1
        for d in range(sub_loops):
            print(isPalindrome(substring[:k+d]))

'''
racar

rac
raca
racar
 aca
 acar
  car
'''

s = "racar"
k = 3

get_substrings(s,k)
```

#### Vowel Substring
```python
# A vowel substring is a substring that only consists of vowels ('a', 'e', 'i', 'o', and 'u') and has all five vowels present in it.

# can have duplicate vowels
# can only contain vowels

'''
Output: 7
cuaieuouac
 uaieuo
 uaieuou
 uaieuoua
  aieuo
  aieuou
  aieuoua
   ieuoua
'''

def vowel_substring(input):
    vl = vowel_list(input)
    k = 5
    #['uaieuoua', 'aaaaa', 'e']
    for v in vl:
        #uaieuoua
        # aieuoua
        #  ieuoua
        for start in range(len(v)-k+1):
            #uaieu
            #uaieuo
            #uaieuou
            #uaieuoua
            # aieuo
            # aieuou
            for end in range(start + k, len(v) + 1):
                print(v[start:end], vowel_check(v[start:end]))
    #print(vl)

def vowel_check(input):
    vowel_list = ['a','e','i','o','u']
    vowel_tracker = {}
    for v in vowel_list:
        vowel_tracker[v]=False
    count = 0
    for i in input:
        if i in vowel_list:
            vowel_tracker[i] = True
    #print(vowel_tracker)
    return all(vowel_tracker.values())

def vowel_list(input):
    vowel_list = ['a','e','i','o','u']

    substring_list = []
    substring = ""

    for i, c in enumerate(input):
       #print(i, len(input)-1, substring)
        if c in vowel_list:
            substring += c
            if substring and i == len(input)-1:
                substring_list.append(substring)
        else:
            if substring :
                substring_list.append(substring)
                substring = ""
    return substring_list

input = "cuaieuouac"
vowel_substring(input)

'''
dict 
    .keys() = [a, i, e, o, u]
    .items() = returns list of tuples [("a":True), ("i":True)]
    .values() = [True, True]
'''
```


# References

[^1]: [https://docs.python-guide.org/writing/tests/](https://docs.python-guide.org/writing/tests/)
