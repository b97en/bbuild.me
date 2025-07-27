---
title: "Journeying through Leetcode with Go: Two Sum"
date: 2025-07-26T17:05:00Z
draft: false
tags: ["golang", "leetcode"]
categories: ["A journey through Leetcode with Go"]
author: "Ben Bullock"
showToc: true
TocOpen: false
weight: 1
cover:
    image: "/images/gopher.jpg"
    alt: "Golang Gopher"
---

### Introduction

Hey, welcome to the first of a series of new posts about Go and Leetcode.

The purpose of this series is to break down the process of working through a variety of Leetcode issues using first principles thinking. I have chosen Go for this because it is the language that I am personally most comfortable with.

---

### The Problem: Two Sum

Here's what we need to do:

Given an array of numbers and a target number, find two numbers in the array that add up to the target. Return their positions (indices) in the array.

Example:

Input: nums = [2, 7, 11, 15], target = 9

Output: [0, 1]

Why? Because nums[0] + nums[1] = 2 + 7 = 9

### Breaking It Down 🔍 

What we're given:

- An array of integers (can be positive, negative, or zero)

- A target number we're trying to reach

What we need to find:

- The positions of two different numbers that add up to our target

- Important notes:

- We need the positions, not the actual numbers

- We can't use the same position twice

- There's always exactly one solution

### The Simple Approach (Brute Force)

Let's start with the most straightforward solution - check every possible pair:

```
gofunc TwoSumBruteForce(nums []int, target int) []int {
    // Check every pair of numbers
    for i := 0; i < len(nums); i++ {
        for j := i + 1; j < len(nums); j++ {
            if nums[i] + nums[j] == target {
                return []int{i, j}
            }
        }
    }
    return nil
}
```

This works! But if we have a large array, it's quite slow because we're checking every single pair.

### A Smarter Solution (Using a Map) 🚀

Here's a clever trick: as we go through the array, we can remember which numbers we've seen and where we saw them.

The idea:

- For each number, calculate what number we'd need to reach the target

- Check if we've already seen that number

- If yes, we found our pair! If no, remember this number for later

Now here's the code:

```
package twosum

// TwoSum finds two numbers that add up to the target
func TwoSum(nums []int, target int) []int {
    // This map remembers: number -> position
    seen := make(map[int]int)
    
    for i, num := range nums {
        // What number would we need to reach the target?
        needed := target - num
        
        // Have we seen this number before?
        if position, found := seen[needed]; found {
            // Yes! We found our pair
            return []int{position, i}
        }
        
        // Remember this number and its position
        seen[num] = i
    }
    
    return nil // This shouldn't happen based on the problem
}
```

### Testing Our Solution 🧪

Always test your code! Here's a simple test:

```
gofunc TestTwoSum(t *testing.T) {
    // Test case 1: Basic example
    nums := []int{2, 7, 11, 15}
    target := 9
    result := TwoSum(nums, target)
    
    // Should return [0, 1] because 2 + 7 = 9
    if result[0] != 0 || result[1] != 1 {
        t.Errorf("Expected [0, 1], got %v", result)
    }
    
    // Test case 2: Numbers in the middle
    nums2 := []int{3, 2, 4}
    target2 := 6
    result2 := TwoSum(nums2, target2)
    
    // Should return [1, 2] because 2 + 4 = 6
    if result2[0] != 1 || result2[1] != 2 {
        t.Errorf("Expected [1, 2], got %v", result2)
    }
}
```

### Why This Solution is Better

The brute force approach:

- Checks every pair: if we have 1000 numbers, that's almost 500,000 checks!

- Time complexity: O(n²) - gets much slower as the array grows

The map approach:

- Goes through the array just once

- Time complexity: O(n) - much faster!

- Trade-off: uses a bit more memory to store the map

### Key Takeaways 📝

- Start simple: It's okay to write the brute force solution first

- Look for patterns: Can we avoid repeated work?

- Use data structures: Maps (hash tables) are great for "have I seen this before?" problems

- Test your code: Always verify with different examples

### Try It Yourself!

Here are some test cases to practice with:

- [3, 3] with target 6

- [-1, -2, -3, -4, -5] with target -8

- [0, 4, 3, 0] with target 0

---

### What's Next?

This "Two Sum" problem introduces a common pattern in programming: using a hash map to remember things we've seen. 

You'll see this pattern again in problems like:

- Finding duplicates
- Checking if two strings are anagrams
- Finding pairs with a specific difference