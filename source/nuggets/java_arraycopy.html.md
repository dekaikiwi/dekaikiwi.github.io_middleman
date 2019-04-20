---
title: Java System.arraycopy
date: 1555767266
---

# Java's System.arraycopy

The below may seem simple to someone has used Java regularly for years, however I'm jumping back in after year or two in the Ruby-sphere and trying to blow the cobwebs out with a few coding challenges on [HackerRank](https://www.hackerrank.com/)

Because Java passes references to the arrays around between methods, we can't alter arrays the context of a method without interfering with the state of the array in other methods.

In cases we need to make some change to an array without altering the initial state of the array. We should use the arraycopy method.

From: [TutorialPoint](https://www.tutorialspoint.com/java/lang/system_arraycopy.htm)

```
public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
```

Now each parameter in my own words...

```
src - The source array that will be copied from
srcPos - The index data will be copied from in the source array
dest - The destination array that dat will be written to (from the source)
destPos - The index that data will be copied to.
length - The length of data that will be copied from srcPos to destPos + (length - 1)
```

I was reviewing code a merge sort implementation I wrote [here](https://gist.github.com/dekaikiwi/35112679b5ccd4209bd3c75c9b5de88b) previously and noticed that when merging sorted arrays I had the following snippet at the end.

```
System.arraycopy(numbers, leftIndex, temp, insertIndex, leftEnd - leftIndex + 1);
System.arraycopy(numbers, rightIndex, temp, insertIndex, rightEnd - rightIndex + 1);
```

In this case insertIndex was set to the same value, which would in theory result in data from the first arraycopy being overwritten.

But upon viewing the while loop that comes before this snippet

```
while (leftIndex <= leftEnd && rightIndex <= rightEnd) {
  leftNum = numbers[leftIndex];
  rightNum = numbers[rightIndex];

  if (leftNum < rightNum) {
    temp[insertIndex] = leftNum;
    leftIndex += 1;
  } else {
    temp[insertIndex] = rightNum;
    rightIndex += 1;
  }

  insertIndex += 1;
}
```

Which means the `length` field for one of the `arraycopy` calls will be zero, preventing any data from being overwritten.

Obvious in hindsight :)

Anyway this nugget is more of less just an exercise to help try and plant the behavior of the `System.arraycopy` method in my head.

There's quite a bit going on in the parameter list, so seems quite easy to forget :/ Seems like the sort of method that needs a googling every time it gets used!
