---
layout: post
title: "Python: Log Parsing"
date: 2024-12-23
tags: ["scripting"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement

What are all the methods to do log parsing? Include the ability to do the following:
* search by specific field.
* sort by specific field
* get count by specific field
* only display result if count is above a specific number

# Nginx access.log sample

```log
46.19.138.234 - - [22/Dec/2024:09:33:40 +0000] - "GET / HTTP/1.1" 200 147 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.85 Safari/537.36 Edg/90.0.818.46" "-" "-" "-" "-" "-"
184.105.139.67 - - [22/Dec/2024:09:36:13 +0000] TLSv1.2 "GET /geoserver/web/ HTTP/1.1" 404 209 "-" "Mozilla/5.0 (Windows NT 6.1; ) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.6261.156 Not(A:Brand/24 YaBrowser/24.4.1.901 Yowser/2.5  Safari/537.36" "-" "-" "-" "-" "-"
184.105.139.67 - - [23/Dec/2024:09:38:24 +0000] TLSv1.2 "GET /.git/config HTTP/1.1" 404 152 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/15.6.1 Safari/605.1.15" "-" "-" "-" "-" "-"
92.255.57.58 - - [24/Dec/2024:09:40:03 +0000] - "GET /actuator/gateway/routes HTTP/1.1" 404 209 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36" "-" "-" "-" "-" "-"
```

# Bash

#### Cut

Basic version
```bash
cat log.txt | cut -d' ' -f1 | uniq -c | sort -r | 
   2 184.105.139.67
   1 92.255.57.58
   1 46.19.138.234
```

Minimum count
```bash
cat log.txt | cut -d' ' -f1 | uniq -c | sort -r | awk '$1 > 1'
   2 184.105.139.67
```

Specific date
```bash
cat log.txt | grep '22/Dec/2024' | cut -d' ' -f1 | uniq -c | sort -r
   1 46.19.138.234
   1 184.105.139.67
``` 

#### Awk

Basic version
```bash
awk '{print $1}' log.txt | uniq -c | sort -r
   2 184.105.139.67
   1 92.255.57.58
   1 46.19.138.234
```

Regex groups
```bash
awk '/^([0-9]+\.[0-9]+\.[0-9]+\.[0-9]+)/ { print $1 }' log.txt
46.19.138.234
184.105.139.67
184.105.139.67
92.255.57.58
```

# Python

#### Custom split
```python
# log-split.py
def main():
	with open("log.txt", "r") as f:
		lines = f.readlines()

	for l in lines:
		#string.split(separator, maxsplit)
		blocks = l.split(" ")
		ip = blocks[0]
		print("IP: ", ip)

if __name__ == "__main__":
	main()
```

#### Regex named groups[^1]
```python
# log-regex.py
import re

#46.19.138.234 - - [22/Dec/2024:09:33:40 +0000] - "GET / HTTP/1.1" 200 147 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.85 Safari/537.36 Edg/90.0.818.46" "-" "-" "-" "-" "-"

def main():
	file = open("output.txt", "w")
	with open("log.txt", "r") as f:
		lines = f.readlines()
		pattern = r'^(?P<ip>\d+\.\d+\.\d+\.\d+)\s-\s-\s\[' \
          r'(?P<date>\d+\/\w+\/\d+:\d+:\d+:\d+\s\+\d+)\]\s[-|\w|\.]+\s\"' \
          r'(?P<verb>\w+)'

		for l in lines:
			match = re.search(pattern, l)
			print(match.group())
			file.writelines(match.group()+"\n")

if __name__ == "__main__":
	main()
```

#### Hashmap[^2]

```python
# log-hashmap.py
import re

#46.19.138.234 - - [22/Dec/2024:09:33:40 +0000] - "GET / HTTP/1.1" 200 147 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.85 Safari/537.36 Edg/90.0.818.46" "-" "-" "-" "-" "-"

def main():
	with open("log.txt", "r") as f:
		hash_map = {}
		lines = f.readlines()
		pattern = r'^(?P<ip>\d+\.\d+\.\d+\.\d+)\s-\s-\s\[' \
          r'(?P<date>\d+\/\w+\/\d+:\d+:\d+:\d+\s\+\d+)\]\s[-|\w|\.]+\s\"' \
          r'(?P<verb>\w+)'

		for l in lines:
			match = re.search(pattern, l)
			if match["ip"] in hash_map:
				hash_map[match["ip"]] += 1
			else:
				hash_map[match["ip"]] = 1
		print("#hash map")
		print(hash_map)
		print()

		for k,v in hash_map.items():
			if v > 1:
				print("#minimal hash map")
				print(k + ", " + str(v))
				print()

		result = {key:value for key,value in hash_map.items() if value > 1}
		print("#oneliner")
		print(result)
		print()

		sorted_hash_map = sorted(hash_map.items(), key=lambda x:x[1], reverse=True)
		print("#sorted hash map")
		print(sorted_hash_map)
		print()		

if __name__ == "__main__":
	main()
```

```bash
python3 log-hashmap.py
#hash map
{'46.19.138.234': 1, '184.105.139.67': 2, '92.255.57.58': 1}

#minimal hash map
184.105.139.67, 2

#oneliner
{'184.105.139.67': 2}

#sorted hash map
[('184.105.139.67', 2), ('46.19.138.234', 1), ('92.255.57.58', 1)]
```

#### SQLite3

```python
# log-sqlite.py
import re
import sqlite3

#46.19.138.234 - - [22/Dec/2024:09:33:40 +0000] - "GET / HTTP/1.1" 200 147 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.85 Safari/537.36 Edg/90.0.818.46" "-" "-" "-" "-" "-"

def main():
	conn = sqlite3.connect('logdb')
	cursor = conn.cursor()
	cursor.execute('''CREATE TABLE IF NOT EXISTS log
	             (log text, count integer)''')

	with open("log.txt", "r") as f:
		hash_map = {}
		lines = f.readlines()
		pattern = r'^(?P<ip>\d+\.\d+\.\d+\.\d+)\s-\s-\s\[' \
          r'(?P<date>\d+\/\w+\/\d+:\d+:\d+:\d+\s\+\d+)\]\s[-|\w|\.]+\s\"' \
          r'(?P<verb>\w+)'

		for l in lines:
			match = re.search(pattern, l)
			if match["ip"] in hash_map:
				hash_map[match["ip"]] += 1
			else:
				hash_map[match["ip"]] = 1

		for k,v in hash_map.items():
			cursor.execute("INSERT INTO log VALUES (?, ?)", (k, v))

	conn.commit()
	conn.close()

if __name__ == "__main__":
	main()
```

```bash
python3 log-sqlite.py
sqlite3 logdb
```

```sql
sqlite> select * from log;
46.19.138.234|1
184.105.139.67|2
92.255.57.58|1
sqlite> select * from log where count > 1;
184.105.139.67|2
sqlite> [ctrl-d]
```

#### Redis[^3] [^4]

```bash
brew install redis
redis-server
python3 log-redis.py
#writes to dump.rdb
```

```python
#log-redis.py
import re
import redis

def main():
	r = redis.Redis()

	with open("log.txt", "r") as f:
		lines = f.readlines()
		pattern = r'^(?P<ip>\d+\.\d+\.\d+\.\d+)\s-\s-\s\[' \
          r'(?P<date>\d+\/\w+\/\d+:\d+:\d+:\d+\s\+\d+)\]\s[-|\w|\.]+\s\"' \
          r'(?P<verb>\w+)'

		for l in lines:
			match = re.search(pattern, l)			
			if r.exists("ip:"+match["ip"]):
				r.incrby("ip:"+match["ip"], 1)
			else:
				r.set("ip:"+match["ip"], 1)

if __name__ == "__main__":
	main()
```

```bash
pip3 install redis
redis-cli
127.0.0.1:6379> PING
127.0.0.1:6379> GET ip:184.105.139.67
"2"
```

#### Redis Sorted Set[^5]

```python
#log-redis-sorted-set.py
import re
import redis

def main():
	r = redis.Redis()

	with open("log.txt", "r") as f:
		lines = f.readlines()
		pattern = r'^(?P<ip>\d+\.\d+\.\d+\.\d+)\s-\s-\s\[' \
          r'(?P<date>\d+\/\w+\/\d+:\d+:\d+:\d+\s\+\d+)\]\s[-|\w|\.]+\s\"' \
          r'(?P<verb>\w+)'

		for l in lines:
			match = re.search(pattern, l)
			ip = match["ip"]			
			if match:
				if r.zscore("ip", ip) is not None:
					r.zincrby("ip", 1, ip)
				else:
					r.zadd("ip", {ip: 1})

if __name__ == "__main__":
	main()
```

```bash
pip3 install redis
redis-cli
127.0.0.1:6379> PING
127.0.0.1:6379> zrange ip 0 1
1) "46.19.138.234"
2) "92.255.57.58"
127.0.0.1:6379> zrange ip 0 2
1) "46.19.138.234"
2) "92.255.57.58"
3) "184.105.139.67"
```

# References

[^1]: [https://safjan.com/python-regex-named-groups/](https://safjan.com/python-regex-named-groups/)

[^2]: A python dictionary, hashtable, and hashmap are synonymous. A quick reminder of how hashmaps work since it's been a hot moment. A key is transformed into a index a.k.a. hashcode using a hashing algorithm. This method enables quick look up without having to search through a list. Hash collisons have to be accounted for, but ideally should be infrequent, see: [https://en.wikipedia.org/wiki/Hash_table#Collision_resolution](https://en.wikipedia.org/wiki/Hash_table#Collision_resolution)

[^3]: Redis stands for Remote Dictionary Service, very similar to a python dictionary

[^4]: [https://realpython.com/python-redis/](https://realpython.com/python-redis/)

[^5]: [https://redis.io/docs/latest/develop/data-types/sorted-sets/](https://redis.io/docs/latest/develop/data-types/sorted-sets/)

