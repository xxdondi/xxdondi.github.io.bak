---
layout: post
title: "Leetcode 46. Permutations"
date: 2023-06-23 16:20:22 +0400
tags: leetcode
---

Let's dive into one of the LeetCode problems which is a great example to learn recursive/backtracking approach to solving algorithmic problems. Also we will outperform 94% of JS solutions on the site 😉 (at the time of writing).

![Solution runtime performance - beats 94% of JS submissions on LeetCode](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ui8vi4yn2tkcp7yzd2tb.png)

## Problem Description

Courtesy of LeetCode:

> Given an array `nums` of distinct integers, return all the possible permutations. You can return the answer in any order.

Our constraints are:

- `1 <= nums.length <= 6`
- `-10 <= nums[i] <= 10`
- All the integers of nums are unique.

## Recursive approach

According to LeetCode itself, the problem is suggested to be solved with recursion/backtracking. For recursion-based solution, we have to figure out what is the _base case_ for recursive calls and what is the _recursive case_. In order for the algorithm to work, we need our recursive case to get to the base case eventually by decreasing the problem size.

Can we apply this approach here?

In this problem all we have is a `nums` array, so probably we need to shrink the array size, until we hit the _base case_, and then work our way up, while keeping in mind the fact that we shrunk the array size, we do not want to lose any data. This is where the backtracking part fits in.

## Defining the base case

Now, the base case is definitely an array of size 1. Why? Permutations of the `[3]` array is just `[[3]]`, we know that for certain, there is no need to do any extra operations here. How do we reduce our initial problem to the base case? We could simply go through each element of the array, removing the first element of the array. If we do that, the recursive case should work it's way to the base case. Then we need to handle the results returned by the base case in the recursive case and return the new result.

## Defining the recursive case

For example, let us take the array of size 2: `nums = [1, 2]`. Is there a way to reduce this input to our base case?

If we remove the first element, we get just `[2]`, this is our base case. In order to complete the permutations array, we should add the missing `1` to the permutations returned by the _base case_, which will get us `[[2, 1]]`. Then we add the missing `1` **back** to the nums array, we get `nums = [2, 1]`. We remove `2`, since this is our first element now and get just `[1]`. This is, again, the base case. Then we add the `2` to the end and get `[1, 2]`. We merge these results and get `[[2, 1], [1, 2]]`.

If it works for the array of size 2, it should work for _an array of any size_.

## Solution

Our theoretical musings lead us to writing something like this:

```javascript
var permute = function (nums) {
  const result = [];
  // - Base case
  if (nums.length === 1) {
    // If there's just 1 element, we simply return a copy
    // of that array
    return [[...nums]];
  }
  // - Recursive case
  // We move through each number in `nums`
  for (let i = 0; i < nums.length; i++) {
    // Remove the first element of `nums`
    const n = nums.shift();
    // Now we have reduced the `nums` size to
    // `nums.length - 1`, this will get us to the
    // base case eventually
    const perms = permute(nums);
    // Traverse through each of the permutations
    // returned by a recursive call
    for (let perm of perms) {
      // We add the number that we shifted earlier
      // to reduce the problem size to the back of
      // the permutation array
      perm.push(n);
      // We add this permutation to the result array
      result.push(perm);
    }
    // Finally, we add the number we shifted back
    // to the array
    nums.push(n);
  }
  // Return the result
  return result;
};
```

This is just enough to solve the problem! Great job on getting to this point in the post.

But we could go the extra mile here.

## Further runtime improvement

What if we make our base case an array of size 2? Figuring out what the permutations of a 2-sized array is pretty straightforward. For example, permutations of `[1, 2]` are `[[1, 2], [2, 1]]`. So if we change our base case to an array of size 2, we could get rid of a few extra recursive calls, and save execution time. Recursive case will do its work as is, no changes needed there.

New base case could look like this:

```javascript
// - Base case
if (nums.length === 2) {
  return [
    [nums[0], nums[1]],
    [nums[1], nums[0]],
  ];
}
```

This should get you to the promised 94%+ runtime result. Enjoy your place on the leaderboard 🥳 .

## Conclusion

LeetCode 46 - Permutations - problem is a great example to learn recursive/backtracking approach. We briefly went through the theoretical basis and applied the same logic to this problem.

Of course, all this is just scratching the surface of the exciting world of algorithm design. Want more depth? Consider reading these books:

- [Grokking Algorithms: Aditya Bhargava](https://www.amazon.com/Grokking-Algorithms-illustrated-programmers-curious/dp/1617292230?&_encoding=UTF8&tag=devtoblog-20&linkCode=ur2&linkId=4d937610eff664c3a2d539fa623f2d35&camp=1789&creative=9325) - great and simple to understand introduction to algorithms;
- [The Algorithm Design Manual: Skiena, Steven S S.](https://www.amazon.com/Algorithm-Design-Manual-Steven-Skiena/dp/1849967202?&_encoding=UTF8&tag=devtoblog-20&linkCode=ur2&linkId=8826c13f2f52be0a1b307471c21565fb&camp=1789&creative=9325) - amazing in-depth guide to algorithm design, it has answers to all your questions on this topic.
