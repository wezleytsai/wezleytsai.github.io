---
layout: post
title: Array Sorting Algorithms, Part 1&#58; Mergesort
---
<!-- links -->

[Marq]: http://marqshort.github.io/

<!-- post -->

This is part 1 of a series of posts going into different array sorting algorithms. I will be looking at mergesort in this one. Check out [Marq's blog] [Marq] for other sorting algorithms, as we'll be looking at different ones during this "sprint".

Mergesort is a divide-and-conquer array sorting algorithm that can be implemented in several ways. Conceptually, the algorithm goes like this:

1. Split the list of _n_ elements into _n_ sublists with 1 element each
2. Repeatedly merge sublists into sorted lists until only 1 list remains

<!--excerpt-->

With an array `[4, 7, 3, 5]`, this is the mergesort procedure:

```
[4, 7, 3, 5] // starting array
[4], [7], [3], [5] // split the array into n lists
[4, 7], [3, 5] // merge lists (1)
[3, 4, 5, 7] // merge lists (2)
```

As you can see, the heavy lifting is done in the **merge lists** step, as that step requires comparing elements and placing them in order.

Here is a recursive implementation of the mergesort algorithm:

```javascript
var mergesort = function(array) {
  if( array.length <= 1 ) return array;

  var first = array.splice(0, Math.floor(array.length/2));

  mergeSort(first);
  mergeSort(array);

  var start = 0;
  outer: while( first.length ) {
    for( var i = start; i < array.length; i++ ) {
      if( first[0] <= array[i] ) {
        array.splice(i, 0, first.shift());
        start = i + 1;
        continue outer;
      }
    }
    array.push(first.shift());
    start = array.length;
  }
}
```

Now let's walk through the algorithm line by line.

This is the base case. If the array is empty or has length of 1, no sorting is required - simply return the original array ontouched.

```javascript
if( array.length <= 1 ) return array;
```

If the array has more than 1 element, then we want to break it down and sort it. The first line simply splices half of the array (or if there are an odd number of elements, it splices `(n + 1)/2` elements). Then, each half is passed into `mergesort`.

```javascript
var first = array.splice(0, Math.floor(array.length/2));

mergesort(first);
mergesort(array);
```

The code above may not look like it is breaking the list down into _n_ sublists, but let's do a step-through for a moment. Each half of the array is passed recursively into `mergesort`, and once it gets there, if they have more than 1 element, the first thing that happens is that they are broken into halves again (and passed recursively into `mergesort` _again_). The result is that original array is broken into sublists of 1 element before the code after these lines get executed.

Now, once we get back to the current stack, we should have two sorted sublists, `first`, and `array`. The only work to do is to merge them to create a single sorted list.

What we will do here is go through each element in the `first` array, and insert them in the appropriate location of the `array` array. That is, if `first` was `[3, 4, 5]`, and `array` was `[2, 6, 8]`, `3` would get inserted at index 1, `4` at 2, and `5` at 3.

First we create a while loop that will continue until `first` is out of elements. Then we loop through `array` and since `first` is sorted, we just need to compare `first[0]` against `array[i]`. If `first[0]` is smaller, then it shoudl be inserted at index `i`. The next element of `first[1]` must be greater than or equal to `first[0]`, so we can change our `start` variable to now point at index `i + 1`, since that is the smallest index that is could be inserted at.

Here, I named the `while` loop so that we can `continue` it once we have inserted an element from `first` into `array` without executing the code after the `for` loop. There are two possible scenarios:

1. `first` is depleted before reaching the end of `array`
2. The end of `array` is reached, but `first` is not empty

Under the second scenario, `continue` does not get executed, so each element in `first` gets pushed into `array` instead. After each `push`, setting `start` equal to `array.length` will skip the `for` loop altogether on each successive iteration of the `while` loop.

This _merge_ step is where most of the work is done. I set up the algorithm this way to reduce space complexity, but there are definitely a multitude of methods to merge two sorted arrays. A helper function can definitely be warranted for this.

```javascript
var start = 0;
outer: while( first.length ) {
  for( var i = start; i < array.length; i++ ) {
    if( first[0] <= array[i] ) {
      array.splice(i, 0, first.shift());
      start = i + 1;
      continue outer;
    }
  }
  array.push(first.shift());
  start = array.length;
}
```

And there you have it, a recursive mergesort algorithm.
