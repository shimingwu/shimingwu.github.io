---
title: BigData(4)
tags:
  - Big Data
date: 2020-11-10 13:26:40
---



This is revision note for NUS master study of CS5425 Big Data Systems for Data Science.

## Locality Sensitive Hashing

- `Jaccard distance/similarity`
- `Shingling`
- `Min-hashing`
- `Locality Sensitive Hashing(LSH)`


**Jaccard Distance/Similarity**

We use Jaccard similarity to define the "distance" betweem two elements.

If we have two **sets**, [1,2,3,4,5,6] and [3,4,6,9,10], there are 3 elements are in intersection: [3,4,6] and there are 8 elements in the union: [1,2,3,4,5,6,9,10], thus, we have:

```
Jaccard similarity = 3/8
Jaccard distance = 1 - Jaccard similarity = 1 - 3/8 = 5/8
```

**Shingling**

Shingling is to convert document into sets.

>A k-shingle (or k-gram) for a document is a sequence of k tokens that appears in the doc.

**The order does matter here and the token can be a character or word or even a feature.** 

```
Example 1:
{Hello world}
k = 3 characters, we will have {Hel, ell, llo, lo_, o_w, _wo, wor, rld}
k = 1 word, we will have {Hello, world}

Example 2:
{abcab}
k = 2, we will have {ab, bc, ca, ab} => remove duplicates => {ab, bc, ca}
Thus, the final result is {ab, bc, ca}
```

Examples on how the to find the similarity between documents.

{% asset_img shingles.png shingles %}
{% asset_img jaccard.png jaccard %}

From the diagram, we can see
```
intersection = 3
union = 6
Jaccard similarity = 3/6
Jaccard distance = 1 - 3/6 = 3/6
```

However, using this method, the computation will be very heavy. 

**Min-hashing**

When a document is very big, it result in a huge amount of shingles. Thus, hashing can be used to compress shingles. 

> Minshaing converts large sets to short signatures, while preserving similarity

**Note: min-hashing is specifically for Jaccard similarity**

The sample of how min-hashing works is shown below.

{% asset_img minhashing.png minhashing %}

The signature matrix taken from the example is the [1,2,2,1]

After calculate multiple rounds of permutations independently, as shown below.

{% asset_img minhashing_3P.png minhashing %}

There are 3 signature matrix:

```
2,1,2,1
2,1,4,1
1,2,1,2
```

The similarity calculation is
```
Column 1 & Column 3:
Column 1 has (1,2)
Column 3 has (1,2,4)
Union: 3 as (1,2,4)
Intersection: 2 as (1,2)
Similarity = 2/3
Distance = 1 - 2/3 = 1/3
(2,4) are the only 1 different. So similarity = 1 / 3 = 0.67
```

**Locality Sensitive Hashing(LSH)**

Find a documents with Jaccard similarity at least s.

## Clustering

