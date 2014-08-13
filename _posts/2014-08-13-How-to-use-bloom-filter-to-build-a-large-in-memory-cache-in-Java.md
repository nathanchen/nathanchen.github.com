---
layout: post
title: "How to use bloom filter to build a large in memory cache in Java"
category: 
tags: ["读文章","算法"]
---
{% include JB/setup %}

- Bloomfilter is designed as an array(A) of m bits. Initially all these bits are set to 0.

##### To add item:

- In order to add any item, it needs to be feed through k hash functions. Each hash function will generate a number which can be treated as a position of the bits array and we shall set that value of that position to 1. For example - first hash function (hash1) on item I produce a bit position x, similarly second and third hash functions produce position y and z.

	A[x] = A[y] = A[z] = 1
	
##### To find item:

- Similar process will be repeated, item will be hashed three times through three different hash functions. Each hash function will produce an integer which will be treated as a position of the array. We shall inspect those x, y, z positions of the bit array and see if they are set to 1 or not. If no, for sure no one ever tried to add this item into bloomfilter, but if all the bits are set, it could be a false positive.

###### To design a good bloomfilter we need to keep track of the following things:

- Good hash functions that can generate wide range of hash values as quickly as possible
- The value of m (size of the bit array) is very important. If the size is too small, all the bits will be set to 1 quickly and false positives will grow largely
- Number of hash functions(k) is also important so that the values get distributed evenly

###### The formula to calculate k and m:

	m = -nlogp / (log2)^2
	where p = desired false positive probability
	
###### The formula to determine k (number of hash functions):
	
	k = m/n log(2)
	where k = number of hash functions, 
	m = number of bits and 
	n = number of items in the filter
	
##### Hashing

- Hashing is an are which affects the performance of bloomfilter. We need to choose a hash function that is effective yet not time consuming.

- We can use two hash functions to generate K number of hash functions.

		First we need to calculate two hash function h1(x) and h2(x)
		Next, we can use these two hash functions to simulate k hash functions of the nature gi(x) = h1(x) + ih2(x), where i can range from {1...k}
	

		long hash64 = ...; // calculate a 64 bit hash function
		// split it in two halves of 32 bit hash values
		int hash1 = (int) hash64; 
		int hash2 = (int) (hash64 >>> 32);
		// Generate k different hash functions with a simple loop
		for (int i = 1; i <= numHashFunctions; i ++)
		{
			int nextHash = hash1 + i * hash2;
		}
		
### Reference:

http://www.javacodegeeks.com/2014/07/how-to-use-bloom-filter-to-build-a-large-in-memory-cache-in-java.html