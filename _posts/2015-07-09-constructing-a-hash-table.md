---
layout: post
title: Constructing a Hash Table
---

<!-- links -->
[wiki]: https://en.wikipedia.org/wiki/Hash_table

<!-- post -->
The beauty of hash tables is obscured in languages such as JavaScript that have built-in data structures for storing key-value pairs. In some languages, hash tables need to be constructed manually using primitive arrays. Aside from the ability to store associative data, hash tables have many performance advantages over other data structures. For example, the value of a key can be retrieved at constant time, independent of how many key-value pairs are in the hash table. Learning about how hash tables work is important, so here is an overview of the inner workings of hash tables.

<!--excerpt-->

## JavaScript Objects ##

In JavaScript, objects can be declared like so to store key-value pairs. The data stored for the key "a" can be retrieved by simply accessing it through `obj`:

```javascript
var obj = {
  a: 1,
  b: 2,
  c: 3
};

console.log(obj['a']); // 1
```

## Arrays ##

Assume that such objects are not available for a moment. How else can this information be stored? This same data can be stored in an array, where each element is a another array, or "tuple" - an ordered list where the order has some significance. In this case, index 0 of the tuple represents the key, and index 1 of the tuple represents the value.

```javascript
var arr = [
  ['a', 1], // this is a tuple
  ['b', 2],
  ['c', 3]
];

console.log(arr[0][1]); // access value of first tuple and retrieve value 1
```

An array such as this one can be created in almost all programming languages. Arrays are stored in computer memory as contiguous memory addresses. To access the second element (at index 1), the computer would only need to find the location of `arr` in memory, and then calculate the memory location of index 1 by adding the memory size of the element at index 0. So if the index is known, the computer can calculate the precise memory location and retrieve it instantly. But what if the index is not known? Only that the value is associated with the key "a"?

The only way to retrieve the associated value is to iterate through the array and check for "a":

```javascript
var value;
for( var i = 0; i < arr.length; i++ ) {
  if( arr[i][0] === 'a') {
    value = arr[i][1];
    break;
  }
}
console.log(value); // 1
```

This process is no longer trivial. If the array had 10,000 elements, the `for` loop would have to (in the worst case) iterate through 10,000 elements and check each tuple before it can return the requested value or verify that it does not exist.

## Hash Functions ##

As mentioned before, if the index of the data is known, it can be retrieved at constant time from an array. That is, the amount of time required to retrieve the data is independent of the size of the array. The array could have 10,000 elements, but the computer can access the data at index 8,157 instantly by doing some math. How can this knowledge by applied to create a data structure that is capable of storing key-value pairs with constant time data retrieval?

A hash table does exactly this. Hash tables store data in an internal array, but has an extra feature that converts the keys (strings) into an index using a **hashing function** to decide _where_ to store the data. Using the example above, a hashing function might do the following:

```javascript
hashingFunction('a', 3); // returns 0
hashingFunction('b', 3); // returns 1
hashingFunction('c', 3); // returns 2
```

The hashing function needs to have the following properties:

- Accepts a string (the key) as an input
- Accepts a size (of internal array) as a second input (so that it can return an index within the bounds of the internal array)
- Needs to be deterministic (the same inputs should always yield the same output)

A simple example of a hashing function might be one that adds up the unicode values of each character of the key and then returns sum modulo size.

To insert a key-value pair such as `['a', 1]`, the `hashingFunction` would convert the key to index 0, and the tuple `['a', 1]` can be stored at that index of the internal array. Similarly, to retrieve the value associated with key "c", the `hashingFunction` would hash "c" into 2, and `arr[2]` can be accessed instantly. The value associated with "c" would be `arr[2][1]`.

This new data structure that _hashes_ keys into indices to store the data in an array is a hash table. However, there is one issue that has yet to be addressed: unless all possible keys are known ahead of time, the hashing function will not be perfect. From the set of all possible strings, the hashing functions will inevitably hash some strings into the same index. This is a _collision_.

## Collision Resolution ##

Many strategies can be used to resolve hash collisions, but one possible solution is to store an array of tuples at each element of the internal storage array. In this example, a key-value pair `['d', 4]` got inserted and the hashing function hashed "d" into 1, which conflicts with `['b', 2]`. We can resolve this conflict by updating the internal array like so:

```javascript
// before inserting ['d', 4]
internalStorage = [
  [['a', 1]], // let's call this a "bucket", which is now an array of "tuples"
  [['b', 2]],
  [['c', 3]]
];

// after inserting ['d', 4]
internalStorage = [
  [['a', 1]],
  [['b', 2], ['d', 4]], // this bucket now has two tuples due to hash conflict
  [['c', 3]]
];
```

To retrieve the value corresponding to the key "d", the hash table will use the hashing function to calculate index 1, then search through the tuples within the bucket at index 1 to find it. Although iterating through the bucket at index 1 may be inefficient (given that the bucket may hold an arbitrarily large number of tuples), a good hash table should store a minimal number of tuples within each bucket. This can be accomplished by using an unbiased hashing function and dynamically resizing the internal array when certain thresholds are crossed.

## Rehashing ##

A hash table that exceeds a certain capacity can get rehashed by following these steps:

1. Create a new internal storage array that is double the original size
2. Using the hashing function (with a different size parameter), re-insert all the key-value pairs into the new array
3. Delete old storage array (or let garbage collection handle it)

Selecting a properly sized array can reduce the number of collisions. A good hashing function should return uniformly distributed indexes that will also keep hash collisions to a minimum. There are no simple methods to avoid hash collisions altogether, so they must be accommodated in some way.

## Conclusion ##

That is the basic concept of a hash table. Although they are great for key-value data storage and retrieval, hash tables are no good for storing data where sequence is important. Hash tables are **unordered** associative arrays. When doing a `for...in` loop, there is no guarantee with respect to the order of the keys. There are many other data structures that each serve specific purposes, and selecting an appropriate data structure can vastly improve performance.
