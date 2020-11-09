---
title: BigData(4)
tags:
  - Big Data
---


This is revision note for NUS master study of CS5425 Big Data Systems for Data Science.

## Locality Sensitive Hashing

- `Jaccard distance/similarity`
- `Shingling`
- `Minhashing/LSH`


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

**Minhashing/LSH`**

When a document is very big, it result in a huge amount of shingles. Thus, hashing can be used to compress shingles. 

> Minshaing converts large sets to short signatures, while preserving similarity


