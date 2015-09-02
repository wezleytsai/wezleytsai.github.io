---
layout: post
title: Array Sorting Algorithms, Part 2&#58; Radix Sort
---
<!-- links -->

[Marq]: http://marqshort.github.io/
[Mergesort]: http://wesleytsai.io/2015/08/08/merge-sort/

<!-- post -->

This is part 2 of a series of posts going into different array sorting algorithms. I will be looking at radix sort in this one. Check out my pal Marq's [blog] [Marq] for other sorting algorithms. Here is [part 1] [Mergesort] of my blog if you missed it.

<!--excerpt-->

## Background

Radix sort is a _non-comparative_ sorting algorithm. A non-comparative sorting algorithm is one that does not compare the whole key to make a sorting decision (by contrast, mergesort _is_ a comparative sorting algorithm). The basic algorithm of radix sort is this:

1. Start with _i_, the least significant digit of each value in the array
2. Sort the values of the array according to the digit
3. Increment _i_ and repeat steps 1 & 2 until the most significant digit has been sorted

As an example, consider the array `[4, 7, 12, 3, 5]`. I padded the numbers with 0s just to make it more obvious which digit it is being sorted against:

```
[04, 07, 12, 03, 05]      // starting array
[12, 03, 04, 05, 07]      // sort based on least significant digist (1s)
[03, 04, 05, 07, 12]      // sort based on second least significant digit (10s)
```

And we are done, because the 10s digit is the most significant digit in this array of numbers. Notice that on the second pass, the order of 3, 4, 5, 7 did not change. The value of the 10s digit for all of these numbers is 0, but their relative order is retained. This can be done by using any **stable** sort for the sorting subroutine.

## Implementation

Here is an implementation of the radix sort:

```javascript
var countsort = function(array, base, d) {
  var i;
  var count = [];
  for (i = 0; i < base; i++) {
    count[i] = 0;
  }

  array.forEach(function(num) {
    count[Math.floor(num / Math.pow(base, d)) % base] += 1;
  });

  for (i = 1; i < count.length; i++) {
    count[i] += count[i - 1];
  }

  var result = [];
  for (i = array.length - 1; i >= 0; i--) {
    var j = count[Math.floor(array[i] / Math.pow(base, d)) % base] - 1;
    result[j] = array[i];
    count[Math.floor(array[i] / Math.pow(base, d)) % base] -= 1;
  }

  for (i = 0; i < array.length; i++) {
    array[i] = result[i];
  }
}

var radixsort = function(array, base) {
  var max = Math.max.apply(array);
  
  for (var d = 0; Math.floor(max / Math.pow(base, d)); d++) {
    countsort(array, base, d);
  }
}
```

Let's first walk through the radix sort function. First, we need to determine the largest number in the array using `Math.max`. As described above, the algorithm stops once the array has been sorted by the most significant digit. I will talk about the `base` in just a bit, for now it's okay to think of all the numbers in base 10. The for loop is iterating through each digit, starting with the 1s (10^0) digit, then 10s (10^1) digit, then the 100s (10^2) digit, until the `max` divided by `base ^ d` is 0. For example, looking at a number such as 78, the for loop will eventually invoke `countsort` with `d = 1` (the most significant digit) and on the next loop when `d` reaches 2, `Math.floor(78/10^2) = 0` and the for loop will exit.

Now let's look at `countsort`, which is doing the heavy lifting of sorting the array by the `d`th digit. This is easiest by looking at a sample array. Let's use `[4, 7, 12, 3, 5]` like above. The first time `countsort` is invoked, with `base = 10` (for now), and `d = 0`, we are comparing the 1s digit of each number.

```javascript
var count = [];
for (i = 0; i < base; i++) {
  count[i] = 0;
}
```

First we construct an array of length = base that is all 0s. In base 10, this would be `[0, 0, 0, 0, 0, 0, 0, 0, 0, 0]`.

```javascript
array.forEach(function(num) {
  count[Math.floor(num / Math.pow(base, d)) % base] += 1;
});
```

Next, we go through each element of the array, extract the `d`th digit, and increment that index of the `count` array. So the 1s digit of our array `[4, 7, 12, 3, 5]` is `[4, 7, 2, 3, 5]` So we end up with `[0, 0, 1, 1, 1, 1, 0, 1, 0, 0]`.

```javascript
for (i = 1; i < count.length; i++) {
  count[i] += count[i - 1];
}
```

Now we go through each element of count, and add to it the count of the previous element. We end up with `[0, 0, 1, 2, 3, 4, 4, 5, 5, 5]`. This is essentially the sort order. Looking at the `d`th digit, if it is a 2 (like it is for the number 12), we see that it goes first (since `count[2] = 1`).

```javascript
var result = [];
for (i = array.length - 1; i >= 0; i--) {
  var j = count[Math.floor(array[i] / Math.pow(base, d)) % base] - 1;
  result[j] = array[i];
  count[Math.floor(array[i] / Math.pow(base, d)) % base] -= 1;
}
```

Again, we go through our original array, and looking at each element's `d`th digit, we set the corresponding index of `result` to that value. Check for yourself that this will work, even if there are duplicates (for example if the array had both 12 and 22, both with their 1s digit being a 2). Also confirm that this countsort is stable, that is, the 12 and 22 will retain their respective order as a result of this sort. Continuing with our example, this gives us `result = [12, 3, 4, 5, 7]`.

```javascript
for (i = 0; i < array.length; i++) {
  array[i] = result[i];
}
```

We want to update `array` to match the sorted numbers, so we simply iterate through and set the values equal to `result`.

In `radixsort`, by repeating the above steps for each digit, the array will end up sorted. Note that this current implementation does not work if the array has negative numbers. However, it is trivial enough to treat the negative sign as an extra "digit". The steps would be: radix sort the non-negatives, radix sort the absolute value of the negatives, reverse the sorted "negatives", add negative signs, and join it with the sorted non-negatives.

## Time Complexity

What is the time complexity of the radix sort algorithm? First we have the function `radixsort`, which iterates through each digit. Then we have `countsort`, which for each digit, will loop through two different arrays - one is the array of length equal to the `base`, and the other is the array of elements, of length `n`. It loops through each a couple of times, but there are no nested loops, so each `countsort` is O(n + k), where _n_ is the number of elements, and _k_ is the base. If we have to loop through `d` digits in `radixsort`, the overall time complexity would be O(d(n+k)).

What is `d`? It depends on what the greatest number in the array is. If we are using base 10 and the greatest number is 100, then `d = log(100) + 1 = 3`. You may have noticed this also depends on the base! If we were instead using base 16, 100 would be represented as 64, and `d = log(100) + 1 = 2` (taking the floor of `log(100)`). We didn't exactly speed up our radix sort algorithm though, since the time complexity is also a function of `k`, which is the base itself. And we arrive at an interesting conclusion: **increasing the base will result in fewer invocations of `countsort`, but each `countsort` will take longer**.

Then the question becomes: what's the ideal `base` to use? Turns out that the `base` that minimizes the time complexity is `n`. This reduces the time complexity to O(nlog(u)), where u is the greatest number in the array. If u is on the order of n^c, where c is a constant, then the time complexity of radix sort turns out to be O(n). That would give us a linear time sorting algorithm so long as the range of numbers being sorted is polynomial in `n`. This is better than comparison based sorting algorithms, but it also further complicates comparisons to other sorting algorithms, since the time complexity of radix sort is dependent on the range of the input.

Further analysis is warranted to fully understand the efficiency of the radix sort, but that's beyond the scope of this post. Hopefully with this, you have a better understanding of the radix sort algorithm.
