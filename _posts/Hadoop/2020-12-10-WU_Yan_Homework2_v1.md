---
layout:     post
title:      "Hadoop Learning Notes - 5"
subtitle:   " \"Hadoop - A-Priori\""
date:       2020.12.10 23:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:

    - Hadoop

---


# 1. Q1

## 1.1. (a) [10 marks]

> *There may be some false positives in the output, i.e., some item pairs that are non-frequent will be included in the final output.*

It is FALSE. 

Firstly, due to the monocity of itemsets(If a set $I$ of items is frequent, then so is every subset of $I$), The reason is simple. Let $J ⊆ I$. Then every basket that contains all the items in $I$ surely contains all the items in $J$. Thus, the count for $J$ must be at least as great as the count for $I$, and if the count for $I$ is at least s, then the count for $J$ is at least $s$. Since $J$ may be contained in some baskets that are missing one or more elements of $I − J$, it is entirely possible that the count for $J$ is strictly greater than the count for $I$.

Secondly, in MapReduce Round 1, it pass through the full dataset in one round and counting all the items that were identified as frequent in the dataset Retain as frequent only those items that were frequent in the dataset. Then in Mapreduce Roud 2, it also parse all pairs in the dataset and finally generates item pairs only in the frequent items, due to the monocity metioned above, the supersets' counts is strictly smaller than the count of single frequent items. In MapReduce Round 3, **it count the pairs in the whole dataset**, so it  will eliminate all false positives.

## 1.2. (b) [10 marks]

> *It may produce some false negatives, i.e., some frequent item pairs are not included in the final output.*

It is FALSE.

Apart from what is metioned above,  in MapReduce Round 1, it make a **complete pass** to find frequent items so that **no possible frequent items are omitted**. And in Round 2, **it generates all item pairs that were supersets of frequent items in such a whole dataset**, according to the monocity, still there will be no possible item pairs being omitted and in Round 3 it aggregate the count and find item pairs that were identified as frequent in the whole, all steps are implemented in a whole dataset instead of a sample so we will have no false negatives.

# 2. Q2

## 2.1.(a) [20 marks] 

> *Implement the A-Priori algorithm to find frequent pairs on a single machine*

### 2.1.1. A-priori_a.py:

```python
#!/usr/bin/env python3

import sys
import itertools

# return the k-round candiates items set
def candidate(l, k):
    candidates_pair = list(itertools.combinations(l, k))
    return candidates_pair
# return the candidates pairs' support dictionary
def count(cand):
    dic_items = {}
    for item_pair in cand:
        dic_items[item_pair] = 0
        for basket_items in whole_items_lists:
            if set(item_pair).issubset(set(basket_items)):
                dic_items[item_pair] += 1
    return dic_items
# read the output from map program and list the whole baskets' items
# then generate the first candidate item sets
dic_first_items_sets = {}
whole_items_lists = []
first_frequent_items = []

for line in f.readlines():
    items_list = line.strip().split()
    whole_items_lists.append(items_list)
    for i in items_list:
        if i not in dic_first_items_sets.keys():
            dic_first_items_sets[i] = 0
            dic_first_items_sets[i] += 1
        elif i in dic_first_items_sets.keys():
            dic_first_items_sets[i] += 1

for item in dic_first_items_sets.keys():
    if dic_first_items_sets[item] / 4984636 >= 0.005:
        first_frequent_items.append(item)
# find the frequent items pairs
k = 2
new_candidates = {}
prior_candidates = candidate(first_frequent_items, k)
dic_candidates = count(prior_candidates)
for pair in dic_candidates.keys():
    if dic_candidates[pair] / 4984636 >= 0.005:
        new_candidates[pair] = dic_candidates[pair]

for item_pair in new_candidates.keys():
    print(item_pair[0], item_pair[1], new_candidates[item_pair])

```

### 2.1.2. Executable bash script

```bash
#!/bin/bash

#point to the local directory
cd /home/1155148594/IERG4300_2

#run the python file
python3 A-priori_a.py > output_a.txt

#sort the results with column three(freqency)
 sort -n -k3 output_a.txt > output_a_sorted.txt
```

### 2.1.3. output_a_sorted file

![image-20201106144301892](https://i.loli.net/2020/11/06/rQOqieS5MF42yhT.png)

and the Top 40 pairs is:

```
('15', 'minutes') 24924
('my', 'same') 24926
('and', 'checked') 24931
('my', 'wife') 24932
('it', 'part') 24934
('actually', 'my') 24935
('get', 'never') 24939
('no', 'other') 24939
('at', 'drinks') 24942
('comes', 'with') 24945
('been', 'some') 24949
('us', 'what') 24954
('for', 'part') 24957
('2', 'this') 24961
('even', 'their') 24962
('look', 'on') 24963
('got', 'ordered') 24964
('of', 'options') 24965
('get', 'way') 24966
('food', 'nnThe') 24968
('I', 'Our') 24969
('waiter', 'we') 24969
('excellent', 'to') 24970
('go', 'only') 24971
('definitely', 'with') 24972
('sat', 'we') 24972
('more', 'when') 24978
('My', 'you') 24980
('because', 'been') 24980
('of', 'ok') 24981
('our', 'who') 24982
('I', 'party') 24983
('any', 'me') 24983
('They', 'when') 24986
('and', 'theyre') 24987
('do', 'food') 24989
('had', 'looked') 24990
('again', 'at') 24991
('know', 'out') 24991
('most', 'you') 24991
```

 

## 2.2. (b) [40 marks]

> *Implement the SON algorithm on MapReduce to find frequent pairs*

### 2.2.1. Mapper_b_1.py

```python
#!/usr/bin/env python3

import sys
import itertools

# return the k-round candiates items set
def candidate(l, k):
    candidates_pair = list(itertools.combinations(l, k))
    return candidates_pair
# return the candidates pairs' support dictionary
def count(cand):
    dic_items = {}
    for item_pair in cand:
        dic_items[item_pair] = 0
        for basket_items in whole_items_lists:
            if set(item_pair).issubset(set(basket_items)):
                dic_items[item_pair] += 1
    return dic_items
# read the output from map program and list the whole baskets' items
# then generate the first candidate item sets
dic_first_items_sets = {}
whole_items_lists = []
first_frequent_items = []

for line in sys.stdin:
    items_list = line.strip().split()
    whole_items_lists.append(items_list)
    for i in items_list:
        if i not in dic_first_items_sets.keys():
            dic_first_items_sets[i] = 0
            dic_first_items_sets[i] += 1
        elif i in dic_first_items_sets.keys():
            dic_first_items_sets[i] += 1
# set the support threshold as 0.005/20 = 0.00025
for item in dic_first_items_sets.keys():
    if dic_first_items_sets[item] / 4984639 >= 0.00025:
        first_frequent_items.append(item)
# find the frequent items pairs
k = 2
new_candidates = {}
prior_candidates = candidate(first_frequent_items, k)
dic_candidates = count(prior_candidates)
for pair in dic_candidates.keys():
    if dic_candidates[pair] / 4984639 >= 0.00025:
        new_candidates[pair] = dic_candidates[pair]

for item_pair in new_candidates.keys():
    print(item_pair[0], item_pair[1], new_candidates[item_pair])

```

### 2.2.2. Reducer_b_1.py

```python
#!/usr/bin/env python3

import sys

candidates_pairs = []
for line in sys.stdin:
    line = line.strip().split()
    tmp = [line[0], line[1]]
    if set(tmp) not in candidates_pairs:
        candidates_pairs.append(set(tmp))

for pair in candidates_pairs:
    print(list(pair)[0], list(pair)[1])
```

### 2.2.3. Mapper_b_2.py

```python
#!/usr/bin/env python3

import sys

f = open("outputphase1.txt")
candidates_item_pairs = []
dic_frequent_pairs = {}
for line in f.readlines():
    line = line.strip().split()
    candidates_item_pairs.append(line)

for line in sys.stdin:
    line = line.strip().split()
    for candidats_pair in candidates_item_pairs:
        if set(candidats_pair).issubset(set(line)):
            dic_frequent_pairs[tuple(candidats_pair)] =  dic_frequent_pairs.setdefault(tuple(candidats_pair), 0) + 1
for pairs in dic_frequent_pairs:
    print(pairs[0], pairs[1], dic_frequent_pairs[pairs])
```

### 2.2.4. Reducer_b_2.py

```python
#!/usr/bin/env python3

import sys

tmp = []
dic_frequent_pairs = {}
for line in sys.stdin:
    line = line.strip().split()
    tmp = [line[0], line[1]]
    dic_frequent_pairs[tuple(tmp)] = dic_frequent_pairs.setdefault(tuple(tmp), 0) + int(line[2])

for pairs in dic_frequent_pairs.keys():
    if dic_frequent_pairs[pairs] / 4984639 >= 0.005:
        print(pairs, dic_frequent_pairs[pairs])
```

### 2.2.5. Executable  bash script

```bash
#!/bin/bash

#point to the local directory
cd /home/1155148594/IERG4300_2

#create input directory in hadoop file system
hadoop fs -mkdir input

#upload the local dataset to the hdfs
hdfs dfs -put yelp_review input

#check whether it is uploaded using following commands
hadoop fs -ls input

#debug the running result locally but the dataset is too large, omit it
cat yelp_review | python3 mapper_b_1.py | sort | python3 reducer_b_1.py

#We need to specify our own streaming file path and all input files and mappper/reducer.
#We also need to specify the path of output files in hdfs archives.
#Due to we use SON algorithm, we need to specify the mappers and reducers
hadoop jar /usr/hdp/2.4.2.0-258/hadoop-mapreduce/hadoop-streaming.jar 
	-D mapred.map.tasks=20
	-D mapred.map.reduce.tasks=10
	-file mapper_b_1.py -mapper mapper_b_1.py 
	-file reducer_b_1.py -mapper reducer_b_1.py 
	-input input/yelp_review 
	-output outputphase1

#during mapreduce running process, you can check the job application status and logs in webUI 
#for example, open  http://dicm2.ie.cuhk.edu.hk:8088/proxy/application_1600915200327_2088/ in WebUI, and check it.

#if mapreduce runs successfully, cat/merge the output file/files on hdfs to a local file
hadoop fs -getmerge outputphase1/part* outputphase1.txt

#check the outputphase1.txt and do the second round mapreduce job
hadoop jar /usr/hdp/2.4.2.0-258/hadoop-mapreduce/hadoop-streaming.jar 
	-D mapred.map.tasks=20
	-D mapred.map.reduce.tasks=10
	-file outputphase1.txt
	-file mapper_b_2.py -mapper mapper_b_2.py 
	-file reducer_b_2.py -mapper reducer_b_2.py 
	-input input/yelp_review 
	-output outputphase2

#cat the outputphase2 file to local machine
hadoop fs -getmerge outputphase2/part* outputphase2.txt

#sort the outputfile
sort -n -k3 outputphase2.txt > output_b2_sorted.txt
```

### 2.2.6. output_b2_sorted

```
('15', 'minutes') 24924
('my', 'same') 24926
('and', 'checked') 24931
('my', 'wife') 24932
('it', 'part') 24934
('actually', 'my') 24935
('get', 'never') 24939
('no', 'other') 24939
('at', 'drinks') 24942
('comes', 'with') 24945
('been', 'some') 24949
('us', 'what') 24954
('for', 'part') 24957
('2', 'this') 24961
('even', 'their') 24962
('look', 'on') 24963
('got', 'ordered') 24964
('of', 'options') 24965
('get', 'way') 24966
('food', 'nnThe') 24968
('I', 'Our') 24969
('waiter', 'we') 24969
('excellent', 'to') 24970
('go', 'only') 24971
('definitely', 'with') 24972
('sat', 'we') 24972
('more', 'when') 24978
('My', 'you') 24980
('because', 'been') 24980
('of', 'ok') 24981
('our', 'who') 24982
('I', 'party') 24983
('any', 'me') 24983
('They', 'when') 24986
('and', 'theyre') 24987
('do', 'food') 24989
('had', 'looked') 24990
('again', 'at') 24991
('know', 'out') 24991
('most', 'you') 24991
```



### 2.2.7. Compare the execution time of (a) and (b)

The job (a) consume 905 minutes while the job (b) only consumes 32 minutes.

## 2.3. (c) [20 Bonus marks]

> *Use the PCY algorithm to filter the candidate pairs in the SON algorithm*

### 2.3.1. Mapper_c_1.py

```python
#!/usr/bin/env python3

import sys
import itertools
from bitmap import BitMap

singleton = {}
whole_item_baskets = []
buckets = {}
bitmap = []
candidate_pairs = []
# define a hash function to hash items to a number
def hash(l1, l2):
    return (l1 + l2) % 100000

# generate single item set with item name converted to item number
for line in sys.stdin:
    line = line.strip().split()
    for i in range(len(line)):
        line[i] = int(''.join([str(ord(line[i][j])) for j in range(len(line[i]))]))
    whole_item_baskets.append(line)
    for item in line:
        singleton[item] = singleton.setdefault(item, 0) + 1
# generate single frequent item set with support threshold 0.005/20 = 0.00025 and delete non-frequent items
for k in list(singleton.keys()):
    if singleton[k] / 4984639 >= 0.00025:
        del singleton[k]
# generate bucket
item_pairs = list(itertools.combinations(singleton.keys(), 2))
for line in whole_item_baskets:
    for pairs in item_pairs:
        if set(pairs).issubset(set(line)):
            hash_value = hash(pairs[0], pairs[1])
            buckets[hash_value] = 1 if hash_value not in buckets else buckets[hash_value] + 1
# initialize a bitmap and compress the bucket into it
bitmap = BitMap(100000)
for key in buckets.keys():
    if buckets[key] / 4984639 >= 0.00025:
        bitmap.set(key)
    else:
        continue
# filter out the candidate pairs
for item in item_pairs:
    if bitmap.test(hash(item[0], item[1])):
        print(item[0], item[1])
```



### 2.3.2. Reducer_c_1.py

```python
#!/usr/bin/env python3

import sys

candidates_pairs = []
for line in sys.stdin:
    line = line.strip().split()
    tmp = [line[0], line[1]]
    if set(tmp) not in candidates_pairs:
        candidates_pairs.append(set(tmp))

for pair in candidates_pairs:
    print(list(pair)[0], list(pair)[1])
```



### 2.3.3. Mapper_c_2.py

```python
#!/usr/bin/env python3

import sys

f = open("outputphase_c1.txt")
candidates_item_pairs = []
dic_frequent_pairs = {}
for line in f.readlines():
    line = line.strip().split()
    candidates_item_pairs.append(line)

for line in sys.stdin:
    line = line.strip().split()
    for candidats_pair in candidates_item_pairs:
        if set(candidats_pair).issubset(set(line)):
            dic_frequent_pairs[tuple(candidats_pair)] =  dic_frequent_pairs.setdefault(tuple(candidats_pair), 0) +\  1
for pairs in dic_frequent_pairs:
    print(pairs[0], pairs[1], dic_frequent_pairs[pairs])

```



### 2.3.4. Reducer_c_2.py

```python
#!/usr/bin/env python3

import sys

tmp = []
dic_frequent_pairs = {}
for line in sys.stdin:
    line = line.strip().split()
    tmp = [line[0], line[1]]
    dic_frequent_pairs[tuple(tmp)] = dic_frequent_pairs.setdefault(tuple(tmp), 0) + int(line[2])

for pairs in dic_frequent_pairs.keys():
    if dic_frequent_pairs[pairs] / 4984639 >= 0.00025:
        print(pairs[0], pairs[1], dic_frequent_pairs[pairs])
```

### 2.3.5. Converter.py

```python
#!/usr/bin/env python3
#generate a dictionary to map item names to item numbers
item_dict = {}
f = open("yelp_review")
r = open("outputphase_c2.txt")
for line in f.readlines():
    line = line.strip().split()
    for i in range(len(line)):
        key = int(''.join([str(ord(line[i][j])) for j in range(len(line[i]))]))
        item_dict[key] = line[i]
# output the item name according to the dictionary
for line in r.readlines():
    line = line.strip().split()
    line[0] = item_dict[line[0]]
    line[1] = item_dict[line[1]]
    print((line[0], line[1]), line[2])
```

### 2.3.6. Executable bash script

```bash
#!/bin/bash

#If local result does not report an error, then run mapreduce programs in Hadoop.
#We need to specify your own streaming file path and all input files and mappper/reducer.
#We also need to specify the path of output files in hdfs archives.
#Due to we use SON algorithm, we need to specify the mappers and reducers
hadoop jar /usr/hdp/2.4.2.0-258/hadoop-mapreduce/hadoop-streaming.jar 
	-D mapred.map.tasks=20
	-D mapred.map.reduce.tasks=10
	-file mapper_c_1.py -mapper mapper_c_1.py 
	-file reducer_c_1.py -mapper reducer_c_1.py 
	-input input/yelp_review 
	-output outputphase_c1

#during mapreduce running process, you can check the job application status and logs in webUI 
#for example, open  http://dicm2.ie.cuhk.edu.hk:8088/proxy/application_1600915200327_2088/ in WebUI, and check it.

#if mapreduce runs successfully, cat/merge the output file/files on hdfs to a local file
hadoop fs -getmerge outputphase_c1/part* outputphase_c1.txt

#check the outputphase_c1.txt and do the second round mapreduce job
hadoop jar /usr/hdp/2.4.2.0-258/hadoop-mapreduce/hadoop-streaming.jar 
	-D mapred.map.tasks=20
	-D mapred.map.reduce.tasks=10
	-file outputphase_c1.txt
	-file mapper_c_2.py -mapper mapper_c_2.py 
	-file reducer_c_2.py -mapper reducer_c_2.py 
	-input input/yelp_review 
	-output outputphase_c2

#cat the outputphase_c2 file to local machine
hadoop fs -getmerge outputphase_c2/part* outputphase_c2.txt

#convert the item number to item name
python3 converter.py > outputphase_c2_converted.txt

#sort the outputfile
sort -n -k3 outputphase_c2_converted.txt > output_c2_sorted.txt
```

### 2.3.7. output_c2_sorted

```
('15', 'minutes') 24924
('my', 'same') 24926
('and', 'checked') 24931
('my', 'wife') 24932
('it', 'part') 24934
('actually', 'my') 24935
('get', 'never') 24939
('no', 'other') 24939
('at', 'drinks') 24942
('comes', 'with') 24945
('been', 'some') 24949
('us', 'what') 24954
('for', 'part') 24957
('2', 'this') 24961
('even', 'their') 24962
('look', 'on') 24963
('got', 'ordered') 24964
('of', 'options') 24965
('get', 'way') 24966
('food', 'nnThe') 24968
('I', 'Our') 24969
('waiter', 'we') 24969
('excellent', 'to') 24970
('go', 'only') 24971
('definitely', 'with') 24972
('sat', 'we') 24972
('more', 'when') 24978
('My', 'you') 24980
('because', 'been') 24980
('of', 'ok') 24981
('our', 'who') 24982
('I', 'party') 24983
('any', 'me') 24983
('They', 'when') 24986
('and', 'theyre') 24987
('do', 'food') 24989
('had', 'looked') 24990
('again', 'at') 24991
('know', 'out') 24991
('most', 'you') 24991
```



### 2.3.8. Compare the execution time of (a) and (b) and (c)

The job (a) consume 905 minutes while the job (b) only consumes 32 minutes and job (c) consumes about 10 minutes.

# 3. Q3

> *The following is a matrix representing three sets, X, Y, and Z, and a universe of five*
> *elements a through e .*
>
> | Row  | X    | Y    | Z    |
> | ---- | ---- | ---- | ---- |
> | a    | 1    | 0    | 1    |
> | b    | 1    | 1    | 0    |
> | c    | 0    | 1    | 1    |
> | d    | 1    | 0    | 0    |
> | e    | 0    | 1    | 1    |
>
> 
>

## 3.1.(a) [3 marks] 

> *Compute the Jaccard similarities of each pair of sets.*

| Pairs | Jaccard Similarities |
| ----- | -------------------- |
| X-Y   | 1/5 = 0.2            |
| X-Z   | 1/5 = 0.2            |
| Y-Z   | 2/4 = 0.5            |

## 3.2. (b) [10 marks]

> *Suppose we create Minhash signatures of length 5 for each of the three*
> *sets X, Y, and Z. The signatures are based on the five cyclic permutations of the*
> *rows. That is, the first permutation uses order abcde, the second uses bcdea, the*
> *third uses cdeab, the fourth deabc, and the fifth eabcd. Give the signature matrix*
> *below.*

Let a, b, c, d, e equals to 1, 2, 3, 4, 5 then signature matrix is:

| Perm.        | X    | Y    | Z    |
| ------------ | ---- | ---- | ---- |
| abcde(12345) | 1    | 2    | 1    |
| bcdea(23451) | 2    | 1    | 1    |
| cdeab(34512) | 1    | 2    | 2    |
| deabc(45123) | 2    | 1    | 1    |
| eabcd(51234) | 1    | 1    | 2    |

## 3.3. (c) [3 marks]

> *What are the estimated Jaccard similarities for each pair of sets according*
> *to the signatures of the sets?*

| Pairs | Estimated Jaccard Similarities |
| ----- | ------------------------------ |
| X-Y   | 1/5 = 0.2                      |
| X-Z   | 1/5 = 0.2                      |
| Y-Z   | 3/5 = 0.6                      |

## 3.4. (d) [4 marks]

> *Consider the use of the Minhash/Locality-Sensitive Hashing (LSH)*
> *scheme to find similar items based on their corresponding columns in the Minhash*
> *signature matrix M . Let n = b · r be the number of rows in M and the n rows of M are*
> *divided into b bands of r rows each. For each band, we hash its portion of each*
> *column to a hash table with k buckets where k is set to be large enough so that the*
> *effect of hash collision is negligible. A pair of items is considered to be a similar-pair*
> *candidate if their corresponding columns are hashed to the same bucket in one or*
> *more bands. For 2 items C1 and C2 have similarity s , namely, their corresponding*
> *signature columns in M actually agree on s fraction of the rows in M . What is the*
> *probability that C1 and C2 will NOT be identified as a similar-pair candidate? Express*
> *your answer in terms of b , r and s ?*

1. The probability the minhash signatures for C1 and C2 agree in any one particular row of the signature matrix is $s$.

2. The probability that the signatures of C1 and C2 agree in all rows of one particular band is $s^r$.

3. The probability that the signatures of C1 and C2 disagree in at least one row of a particular band is $1-s^r$.

4. The probability that the signatures of C1 and C2 disagree in at least one row of each of the bands is $(1-s^r)^b$.

Therefore, the probability that C1 and C2 will NOT be identified as a similar-pair candidate is $(1-s^r)^b$.



