---
layout:     post
title:      "Hadoop Learning Notes - 3"
subtitle:   " \"Hadoop - Community Detection for Similar Users\""
date:       2020.10.09 23:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 作业
    - Hadoop





---

> *"Keep Learning Hadoop"*

# Homework 1 of IERG4300-Big Data Technology and Applications 

# 1. Homework A

> *For EVERY user, recommend the person with the maximal number of common followees in the medium-sized dataset [2]. If multiple people share the same number, randomly pick one of them.*

## 1.1. mapper.py

```python
#!/usr/bin/env python3

import sys

# input comes from STDIN (standard input)

for line in sys.stdin:
    # remove leading and trailing whitespace
    line = line.strip()
    # split the line into words
    followPair = list(line.split())
    # # increase counters
    celebrity = followPair[0]
    fans = followPair[1]
    # Switch the output format and make follower the key, followee the value
    print('%s\t%s' % (fans, celebrity))

# --coding:utf-8 -``    
```

## 1.2. reducer_a.py

```python
#!/usr/bin/env python3

from operator import itemgetter
import sys
from collections import Counter

same_follwee_list = []
last_follower = None
d = {}
# def get_order_dict_N(_dict, N):
#    result = Counter(_dict).most_common(N)
#    d = {}
#    for k,v in result:
#        d[k] = v
#    return d

for line in sys.stdin:
    line = line.strip()
    cur_follower, cur_followee = line.split()
    cur_follower = int(cur_follower)
    cur_followee = int(cur_followee)

    if cur_follower == last_follower:
        same_follwee_list.append(cur_followee)
    else:
        if last_follower:
            d[last_follower] = []
            d[last_follower] += same_follwee_list

        last_follower = cur_follower
        same_follwee_list = [cur_followee]

if cur_follower == last_follower:
    d[last_follower] = []
    d[last_follower] += same_follwee_list

for cur_follower in d.keys():
    d2 = {}
    max_common_followees = 0
    most_common_follower = None
    for otherfollower in list(d.keys()):
        if otherfollower != cur_follower:
            commonfollowees = list(set(d[cur_follower]).intersection(set(d[otherfollower])))
            if len(commonfollowees) >= max_common_followees:
                max_common_followees = len(commonfollowees)
                most_common_follower = otherfollower
    print(str(cur_follower) + ": " + str(most_common_follower))
    
# --coding:utf-8 -``
```

## 1.3. Mapreduce command

```
hadoop jar /usr/hdp/2.4.2.0-258/hadoop-mapreduce/hadoop-streaming.jar -D mapred.map.tasks=49 -D mapred.reduce.tasks=49 -file mapper.py -mapper mapper.py -file reducer_a.py -reducer reducer_a.py -input input/twitter_combined.txt -output outputHWA
```

## 1.4. output_HWA_full_results

**The first 20 lines of it:**

![image-20201009151846421](https://i.loli.net/2020/10/09/NTgMPGiyp5Jknoa.png)

**The last 20 lines of it:**

![image-20201009151906550](https://i.loli.net/2020/10/09/zIhXBu1Q5J6gwHA.png)

## 1.5. output_HWA_filtered

### 1.5.1. Filter it using a simple python script locally.

```python
f = open("output_HWA")

for line in f.readlines():
    tmp_line = line.strip().split(":")
    user = tmp_line[0]
    if user[-5 : ] == "48594":
        print(line)
```

### 1.5.2. Output

```
14348594: 16956955
```

# 2. Homework B

> *Find the top K (K=3) most similar people of EVERY user for the medium-sized dataset in [2]. If multiple people have the same similarity, randomly pick three of them.*

## 2.1. mapper.py

```python
#!/usr/bin/env python3

import sys

# input comes from STDIN (standard input)

for line in sys.stdin:
    # remove leading and trailing whitespace
    line = line.strip()
    # split the line into words
    followPair = list(line.split())
    # # increase counters
    celebrity = followPair[0]
    fans = followPair[1]
    # Switch the output format and make follower the key, followee the value
    print('%s\t%s' % (fans, celebrity))

# --coding:utf-8 -``    
```

## 2.2. reducer_b.py

```python
#!/usr/bin/env python3

from operator import itemgetter
import sys
from collections import Counter

same_follwee_list = []
last_follower = None
d = {}
def get_order_dict_N(_dict, N):
    result = Counter(_dict).most_common(N)
    d = {}
    for k,v in result:
        d[k] = v
    return d

for line in sys.stdin:
    line = line.strip()
    cur_follower, cur_followee = line.split()
    cur_follower = int(cur_follower)
    cur_followee = int(cur_followee)

    if cur_follower == last_follower:
        same_follwee_list.append(cur_followee)
    else:
        if last_follower:
            d[last_follower] = []
            d[last_follower] += same_follwee_list

        last_follower = cur_follower
        same_follwee_list = [cur_followee]

if cur_follower == last_follower:
    d[last_follower] = []
    d[last_follower] += same_follwee_list

for follower in d.keys():
    d2 = {}
    for otherfollower in list(d.keys()):
        if otherfollower != follower:
            commonfollowees = list(set(d[follower]).intersection(set(d[otherfollower])))
            allfollowees = list(set(d[follower]).union(set(d[otherfollower])))
            similarity = len(commonfollowees) / len(allfollowees)
            d2[str(otherfollower)] = similarity
    d3 = get_order_dict_N(d2, 1)
    print(str(follower) + ": " + " ".join(list(d3.keys())))
    
# --coding:utf-8 -``
```

## 2.3. Mapreduce command

```
hadoop jar /usr/hdp/2.4.2.0-258/hadoop-mapreduce/hadoop-streaming.jar -D mapred.map.tasks=49 -D mapred.reduce.tasks=49 -file mapper.py -mapper mapper.py -file reducer_b.py -reducer reducer_b.py -input input/twitter_combined.txt -output outputHWB
```

## 2.4. output_HWB_full_result

**The first 20 lines of the output:**

![image-20201009155337454](https://i.loli.net/2020/10/09/Zr3BO1vwtmebF2q.png)

**The last 20 lines of the output:**

![image-20201009155517006](https://i.loli.net/2020/10/09/yflBALj6NHJFZT2.png)

## 2.5. output_HWB_filtered

### 2.5.1. Filter it using a simple python script locally.

```python
f = open("output_HWB")

for line in f.readlines():
    tmp_line = line.strip().split(":")
    user = tmp_line[0]
    if user[-5 : ] == "48594":
        print(line)
```

### 2.5.2. Output

```
14348594: 17293897 14326840 16956955
```

# 3. Homework C

> *Besides the number of similar users for a target user, say A, sometimes we also want to know the IDs of the common followees shared between A and its similar users. In our example, for User A, if B is the similar user (TopK in Q1.b) to A, the desirable*
> *output should be:*
> *A: B, {C, E}, check_sum for the IDs of the common_followees………(F1)*

## 3.1. mapper.py

```python
#!/usr/bin/env python3

import sys

# input comes from STDIN (standard input)

for line in sys.stdin:
    # remove leading and trailing whitespace
    line = line.strip()
    # split the line into words
    followPair = list(line.split())
    # # increase counters
    celebrity = followPair[0]
    fans = followPair[1]
    # Switch the output format and make follower the key, followee the value
    print('%s\t%s' % (fans, celebrity))

# --coding:utf-8 -``    
```

## 3.2. reducer_c.py

```python
#!/usr/bin/env python3

from operator import itemgetter
import sys
from collections import Counter

same_follwee_list = []
last_follower = None
d = {}
def get_order_dict_N(_dict, N):
    result = Counter(_dict).most_common(N)
    d = {}
    for k,v in result:
        d[k] = v
    return d

for line in sys.stdin:
    line = line.strip()
    cur_follower, cur_followee = line.split()
    cur_follower = int(cur_follower)
    cur_followee = int(cur_followee)

    if cur_follower == last_follower:
        same_follwee_list.append(cur_followee)
    else:
        if last_follower:
            d[last_follower] = []
            d[last_follower] += same_follwee_list

        last_follower = cur_follower
        same_follwee_list = [cur_followee]

if cur_follower == last_follower:
    d[last_follower] = []
    d[last_follower] += same_follwee_list

for follower in d.keys():
    d2 = {}
    for otherfollower in list(d.keys()):
        if otherfollower != follower:
            commonfollowees = list(set(d[follower]).intersection(set(d[otherfollower])))
            allfollowees = list(set(d[follower]).union(set(d[otherfollower])))
            similarity = len(commonfollowees) / len(allfollowees)
            d2[str(otherfollower)] = similarity
    d3 = get_order_dict_N(d2, 3)

    for similar_user in d3.keys():
        common_followees_set = set(d[follower]).intersection(set(d[int(similar_user)]))
        print(str(follower) + ": " + str(similar_user) + ", " + \
              str(common_followees_set) + ", " + str(sum(common_followees_set)))

    
# --coding:utf-8 -``
```

## 3.3. Mapreduce command

```
hadoop jar /usr/hdp/2.4.2.0-258/hadoop-mapreduce/hadoop-streaming.jar -D mapred.map.tasks=49 -D mapred.reduce.tasks=49 -file mapper.py -mapper mapper.py -file reducer_c.py -reducer reducer_c.py -input input/twitter_combined.txt -output outputHWC
```

## 3.4. output_HWC_full_result

**The first 20 lines of the output:**

![image-20201009161055688](https://i.loli.net/2020/10/09/OfpsZwzLU5cPjr4.png)

**The last 20 lines of the output:**

![image-20201009161119440](https://i.loli.net/2020/10/09/9MQOE13jfcShgDT.png)

## 3.5. output_HWC_filtered

### 3.5.1. Filter it using a simple python script locally.

```python
f = open("output_HWC")

for line in f.readlines():
    tmp_line = line.strip().split(":")
    user = tmp_line[0]
    if user[-5 : ] == "48594":
        print(line)
```

### 3.5.2. Output

```
14348594: 14326840, {3840, 13055232, 1010181, 16133, 15846407, 14632458, 7668362, 64508047, 50338197, 794136, 856731, 3829151, 13348, 767396, 7846, 5768872, 14331688, 14170924, 809010, 14807093, 2154171, 13649212, 1239741, 14596798, 15221950, 14134204, 15827269, 1885511, 744903, 917321, 2363721, 143303, 13809612, 10314702, 92623, 774096, 5796562, 1077971, 15575251, 14325471, 608993, 162442852, 732773, 610533, 2148071, 7093352, 14500709, 824168, 16970864, 6449392, 10165232, 817268, 11435642}, 602679293	

14348594: 16956955, {57917441, 8067082, 2735631, 22272553, 242923569, 15200820, 40381496, 49836089, 17732153, 1084, 15120464, 17419360, 2317921, 14217830, 3359851, 16416378, 15151229, 22737021, 18746498, 7076492, 678033, 9212052, 16848539, 72952988, 17667742, 19334304, 11242152, 2779821, 15666380, 34556109, 32423136, 18151650, 15300838, 767, 19197186, 926981, 20104965, 16856857, 42583325, 14335262, 26868515, 749863, 2100521, 131657519, 26882352, 73448241, 66513205, 17105214, 14498622, 17853760, 1183041, 1385281, 139162440, 6539592, 5984072, 15773009, 18720595, 130922325, 14224219, 8946022, 17815912, 16404844, 8126322, 12382582, 20958080, 21600144, 111759249, 1835411, 22722452, 16906137, 35206553, 36732828, 3829151, 14990751, 15583650, 6572452, 17825703, 14414761, 24493, 14630319, 18104752, 10481072, 129939890, 14743985, 22453, 15874998, 7081402, 22954430, 47461824, 13809612, 5538252, 17612764, 14148063, 71273952, 10213, 27721192, 23641580, 18708466, 22461427, 8914942}, 2591795520	

14348594: 17293897, {16129920, 57917441, 16879362, 45607170, 17156994, 22080646, 14529929, 33423, 150802319, 24431892, 14326805, 19288213, 55912983, 43382040, 18622869, 14437914, 71028123, 74625821, 14677919, 435926182, 313091751, 235592999, 11447852, 18104752, 130617778, 14478260, 40381496, 20351289, 132948409, 23483324, 4119741, 12193342, 14293310, 14278978, 1106501, 14133447, 139162440, 6442312, 79797834, 586, 22766925, 87031118, 14511951, 18655567, 165511377, 15329102, 18720595, 6095, 180540375, 14251225, 85822297, 27644378, 13783772, 149103331, 31203, 17431654, 21017448, 34394473, 115710058, 23235056, 14246001, 13483762, 22461427, 87330803, 15938936, 271725689, 39436030}, 3843945014	
```

# 4. Homework D

> *Run part (a) for the medium dataset multiple times while modifying the number of mappers and reducers for your MapReduce job(s) each time. You need to examine and report the performance of your program for at least 4 different runs. Each run should use a different combination of number of mappers and reducers. For each run, performance statistics to be reported should include: (i) the time consumed by the entire MapReduce job(s) and the maximum, minimum and average time consumed by (ii) mapper and reducer tasks and (iii) Tabulate the time consumption for each MapReduce job and its*
> *tasks. One example is given in the following table. Explain your observations.*

## 4.1. Table of results

| Mapper num | Reducer num | Max mapper time | Min mapper time | Avg mapper time | Max reducer time | Min reducer time | Avg reducer time | Total job |
| ---------- | ----------- | --------------- | --------------- | --------------- | ---------------- | ---------------- | ---------------- | --------- |
| 49         | 49          | 11s             | 2s              | 5.6s            | 24s              | 11s              | 16.9s            | 36s       |
| 5          | 49          | 5s              | 3s              | 5s              | 21s              | 9s               | 13.6s            | 30s       |
| 5          | 37          | 7s              | 5s              | 6.5s            | 29s              | 14s              | 20.1s            | 42s       |
| 10         | 49          | 7s              | 3s              | 5.1s            | 22s              | 9s               | 13.3s            | 31s       |
| 10         | 26          | 12s             | 4s              | 10.2s           | 52s              | 28s              | 36.8s            | 67s       |
| 10         | 40          | 15s             | 3s              | 12.5s           | 28s              | 20s              | 23.3s            | 38s       |
| 1049       | 49          | 22s             | 1s              | 5s              | 302s             | 213s             | 264s             | 331s      |
| 100        | 5           | 11s             | 2s              | 6.11s           | 933s             | 792s             | 884s             | 944s      |

## 4.2. Conclusion

**Due to the complexity of my `reducer_a.py` is much higher than `mapper.py`, I need to set more reducer tasks to reduce the total time consumed. And it is unnecessary to launch many mapper tasks, which will not have a positive effect.**

# 5. Homework E

> *Find the TOP 3 (=K) most similar people and the list of common followees for each user in the large dataset in [3] using the format of Q1(b). (Hints: To reduce memory consumption of your programme, you may consider to use the composite*
> *key design pattern and secondary sorting techniques as discussed in [7] and [8]).*

## 5.1. MapreduceJob

I just used the same `mapper.py` and `reducer_c.py` as is shown above, and summit the following command to Hadoop Cluster:

```
~/HW1$ hadoop jar /usr/hdp/2.4.2.0-258/hadoop-mapreduce/hadoop-streaming.jar -D mapred.map.tasks=20 -D mapred.reduce.tasks=150 -file mapper.py -mapper mapper.py -file reducer_c.py -reducer reducer_c.py -input input/gplus_combined.txt -output outputHWE
```

Then I find a little different from other jobs, the `Resource Manager` automatically add other nodes on this job:

![image-20201009170635740](https://i.loli.net/2020/10/09/14gwQEnRUcZoApe.png)

Anyway, this job only consumed 117 seconds:

![image-20201009170755290](https://i.loli.net/2020/10/09/ZO7bUkWBMzqc1fx.png)

## 5.2. Check the output

### 5.2.1. Merge the output files to local machine.(the same as the last few jobs, which I have omitted to state)

```
hadoop fs -getmerge outputHWE output_HWE
```

### 5.2.2. Check the output file

![image-20201009173449035](https://i.loli.net/2020/10/09/zD52BPcL7msQg4q.png)

### 5.2.3. Filter the output file

**Filter Script:**

```python
f = open("output_HWE")

for line in f.readlines():
    tmp_line = line.strip().split(":")
    user = tmp_line[0]
    if user[-5 : ] == "48594":
        print(line)
```

**Output:**

```
101846564375625648594: 110581389615206127584, {101350819212767050043, 108652029511994590330, 107711981740147507868, 105902872654387010050, 102522500090947503238, 106542106648136996331, 116845976432088781351, 101626577406833098387}, 851154863697302537598	

101846564375625648594: 109144827483985204411, {101626577406833098387, 115290690553545888015, 101350819212767050043, 108652029511994590330, 113848938016044308953, 100999054273222870046, 106409865193655273598, 107711981740147507868, 105902872654387010050, 116812976256657521981, 102522500090947503238, 113699708767960396706, 102012612770434256811, 106542106648136996331, 101542361632933122829, 109721710332650526284, 114979500386341603694}, 1829626305448659525164	

101846564375625648594: 103716366627982837466, {115290690553545888015, 109040281510266023760, 104276294792480517398, 100999054273222870046, 113848938016044308953, 107711981740147507868, 106409865193655273598, 114097496219861321469, 105902872654387010050, 116812976256657521981, 118069281985350505214, 112377172384039230241, 102012612770434256811, 106542106648136996331, 101542361632933122829, 114062284545761674313, 109721710332650526284, 101626577406833098387}, 1960344558916407653548	

103558688401827648594: 108427139667434015678, {116534900336716176887, 114057312782572235764, 102340116189726655233, 104626296732158879040}, 437558626041173946924	

103558688401827648594: 100337076202382471967, {101268667866741194829, 109146268902786795235, 118052241402382187216, 116584112681087621706, 102340116189726655233}, 547391407042724454219	

103558688401827648594: 118052241402382187216, {113279794098002104915, 102779774343332622175, 108258708238715051359, 114204547696242413789, 106355644622842216417, 104482470639293494113, 115304442352141608283, 109146268902786795235, 114239019095523962467, 115156877578540641253, 107575507172424264297, 117020656025245358698, 114648167265033384941, 104410565826697099763, 106672354193358676981, 114057312782572235764, 116534900336716176887, 116690766770197620217, 110604101580019940093, 102340116189726655233, 104441943329630805886, 106192190006951379201, 111258641113358659200, 100255479358995396743, 112344687375792379398, 103876221190174441108, 104227162571527308949, 109679468628296768532, 109218397058848562322, 112870754718808962453, 113270197490827805845, 115657027275587830165, 110952817247587568540, 112447053770192628510, 116330759361962343709, 104308826533824927649, 108685174006449246113, 118369296774508770337, 103570613990763043756, 109489178494093125163, 116889391710617193514, 117309116348535379889, 103913990086430896183, 100942277385503843642, 100767348360252393786, 101445766182184837049, 102022434129636807230, 104626296732158879040, 107985192097130438595, 101268667866741194829, 109871899210962066891, 110911634040796480076, 116584112681087621706, 106364829450438152911}, 5902110842290070461800	
```

