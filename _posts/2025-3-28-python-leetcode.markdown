---
layout: post
title: "Python: Leetcode Practice II"
date: 2025-3-28
tags: ["python"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Solve some Leetcode problems using Python: [https://leetcode.com/problemset/](https://leetcode.com/problemset/)

# Leetcode
Private Repo: [https://github.com/JacksonKuo/scripts-leetcode](https://github.com/JacksonKuo/scripts-leetcode)

* [https://leetcode.com/problems/valid-palindrome/](https://leetcode.com/problems/valid-palindrome/)
* [https://leetcode.com/problems/palindrome-partitioning/](https://leetcode.com/problems/palindrome-partitioning/)
* [https://leetcode.com/problems/2sum/](https://leetcode.com/problems/2sum/)
* [https://leetcode.com/problems/3sum/](https://leetcode.com/problems/3sum/)
* [https://leetcode.com/problems/simplify-path/](https://leetcode.com/problems/simplify-path/)

#### Palindrome
```python
import re

def palindrome(str):
	if len(str) < 1:
		return False

	cleaned = clean(str)
	print(cleaned)
	return check(cleaned)

def check(str):
	length = len(str)
	iterate = int(len(str) / 2)
	#print(iterate)

	for i, l in enumerate(str):
		top = str[i]
		bottom = str[length-i-1]
		if top != bottom:
			return False
	else:
		return True

def clean(str):
	lowered = str.lower()
	return re.sub(r"[^a-z]","",lowered)

str = "racar"

print(palindrome(str))
```

#### Palindrome Substring
```python
import re

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
	length = len(s)
	slide = length - k + 1
	print("slide: ",slide)
	for i in range(slide):
		substring = s[i:i+k]
		print(substring)
		print(isPalindrome(substring))

s = "racar"
k = 3

get_substrings(s,k)
```

#### Two Sum
```python
#Input: nums = [2,7,11,15], target = 9
#Output: [0,1]

def twosum(nums, target):
    print(nums)
    print(target)
    #iterate array

    for i, n in enumerate(nums):
        search = target - n
        #print(search)
        copy_nums = nums[:i] + [None] + nums[i+1:]
        #print(copy_nums)

        if search in copy_nums:
            pos = copy_nums.index(search)
            print([nums[i],nums[pos]])
            return([i,pos])

nums = [2,7,11,15]
target = 9
print(twosum(nums, target))
print("")

nums = [3,2,4]
target = 6
print(twosum(nums, target))

nums = [3,3]
target = 6
print(twosum(nums, target))
```

#### Three Sum
```python
input = [1,2,3,4,5]

for i in range(len(input)):
	for j in range(i+1,len(input)):
		for k in range(j+1,len(input)):
			print(i,j,k)
```

> Solve using O(n^2)

#### Simplify Path + CD
```python
def simplifyPath(pwd, cd):
	print("pwd:", pwd, "cd:", cd)

	full_path = pwd.split("/")

	cd_list = cd.split("/")
	for i, l in enumerate(cd_list):
		
		if cd.startswith("/"):
			return(cd)
		if l.isalpha():
			print("push: ",l)
			full_path.append(l)
		if l == ".." and len(full_path) > 1:
			print("pop: ",l)
			full_path.pop()
		if l == ".":
			continue

	return stringify(full_path)

def stringify(path):
	full = ""
	for p in path:
		full += p
		full += "/"
	return full

print("result: ",simplifyPath("/bar", "/bar/"))
print("")
print("result: ",simplifyPath("/bar", "/foo"))
print("")
print("result: ",simplifyPath("/bar", "zzz"))
print("")
print("result: ",simplifyPath("/bar", "../a"))
print("")
print("result: ",simplifyPath("/bar", "a/./b"))
print("")
print("result: ",simplifyPath("/bar", "a/../b/../c"))
print("")
print("result: ",simplifyPath("/bar", "../../../../../"))
```

# Logging
```python
import logging

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.basicConfig(level=logging.DEBUG)
logger.info('')
```

# Tests[^1] [^2]
```python
import unittest

def fun(x):
    return x + 1

class MyTest(unittest.TestCase):
    def test(self):
        self.assertEqual(fun(3), 4)

#python3 -m unittest discover
```

# Python Refresher
CodeAcademy has a nice sandbox to refresh on the syntax. Python 3 is behind the paywall, but Python 2 is free and essentially the same:[^3]
* python syntax
* strings
* conditional and control flow
* functions
* lists & dictionaries
* lists and functions
* loops
* advanced topics
* classes
* file input and output

#### Python 2 vs Python 3[^4]

| Python2 | Python3 |
|---|---|
| 5 / 2 = 2  | 5 / 2 = 2.5 | 
| strings = ascii | string = unicode | 
| `except Exception, e:` | `except Exception as e` | 
| `raw_input()` | `input()` | 
| `long`/`int` type | `int` type = unlimited precision | 
| `xrange` | `range()` |
| `print x` | `print(x)` |

#### Syntax Reminders[^5]
* use underscore for python var names
* booleans are capitalized, `True`/`False`
* `fifth_letter = "MONTY"[4]`
* string functions
    * `len()`
    * `lower()`
    * `upper()`
    * `str()`
    * `print("%s %s" % ("a", "b"))`
* datetime
    * 
        ```
        from datetime import datetime
        print datetime.now()
        year = now.year
        month = now.month
        day = now.day
        ```
* lists
    * array
    * `empty_list = []`
    * 
        ```
        letters = ['a', 'b', 'c']
        letters.append('d')
        list_length = len(letters)
        ```
    * 
        ```
        abc = ["a", "b", "c"]
        index = abc.index("b")
        ```
    * 
        ```
        start_list = [5, 3, 1, 2, 4]
        square_list = []
        for s in start_list:
            square_list.append(s ** 2)
        square_list.sort()
        ```
    * append is for list
    * pop, removes the index, 1
        * `n.pop(1)`
    * remove, removes value, not the index
        * `n.remove(1)`
    * del, removes the index n[1]
        * `del(n[1])`
* `"`  and `'` are functionally the same in python
* dictionaries
    * hashmap
    * `empty_dict = {}`
    * loop a dict returns the key
    *
        ```
        backpack = ['a', 'b', 'c', 'd']
        backpack.remove('c')
        ```
    * 
        ```
        from datetime import datetime
        now = datetime.now()

        print('%02d/%02d/%04d' % (now.month, now.day, now.year))
        ```
    * don't call the input string `string`, if you use a string()
* `range()`
    * generates a list
    * range(start, stop, step)
* concatenate lists
    * 
        ```
        a = [1, 2, 3]
        b = [4, 5, 6]
        print a + b
        ```
* `for index, l in enumerate(list)`
* `index()` needs a try and except, if no index found, error
* 
    ```
    while:
        ...
    else:
    ```
* 
    ```
    for:
        ...
    else:
    ```
* print uses `,` not `+`
    * `print("a" + "b")`
    * `print("a","b")`
    * a comma after `print()`, will keep printing on the same line
* `zip()`
    * iterate over two lists at once
    * `for a, b in zip(list_a, list_b):`

* built-in data structures[^6]

| built-in data structures | Dict | Set | List | Tuple |
|---|---|---|---|---|---|
| syntax | {} | {} | [] | () | 
| duplication | N | N | Y | Y | 
| empty | d = {} | a = set() | l = [] | t = () | 
| sort | ordered (3.6+) | unordered | ordered | ordered |
| change | mutable | mutable | mutable | immutable |


# References

[^1]: [https://docs.python-guide.org/writing/tests/](https://docs.python-guide.org/writing/tests/)

[^2]: [https://docs.python.org/3/library/unittest.html](https://docs.python.org/3/library/unittest.html)

[^3]: [https://www.codecademy.com/enrolled/courses/learn-python](https://www.codecademy.com/enrolled/courses/learn-python)

[^4]: [https://www.datacamp.com/blog/python-2-vs-3-everything-you-need-to-know](https://www.datacamp.com/blog/python-2-vs-3-everything-you-need-to-know)

[^5]: [https://www.codecademy.com/learn/learn-python](https://www.codecademy.com/learn/learn-python)

[^6]: [https://www.geeksforgeeks.org/differences-and-applications-of-list-tuple-set-and-dictionary-in-python/](https://www.geeksforgeeks.org/differences-and-applications-of-list-tuple-set-and-dictionary-in-python/)