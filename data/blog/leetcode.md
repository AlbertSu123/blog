# Array

TLDR: put everything into a map, then solve

## [Contains Duplicate](https://leetcode.com/problems/contains-duplicate/)

Easy - use a map

## [Valid Anagram](https://leetcode.com/problems/valid-anagram/)

Easy, use a map

## [Two sum](https://leetcode.com/problems/two-sum/)

Easy, use a map

## [Group Anagrams](https://leetcode.com/problems/group-anagrams/)

Easy, just use a nested map

## [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)

Not super easy, but you need to use a nested map and to iterate backwards through the map

## [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/)

Needed to submit this a few times, ran into issues with array indexes, which were solved by drawing things out on a piece of paper.
Also, the optimal strategy for me on this problem was to solve it first with brute force(use two hash maps), then it was obvious what work was repeated such that we could do it without any extra space.

## [Valid Sudoku](https://leetcode.com/problems/valid-sudoku/)

Easy, but requires a bit of coding and thinking through how to iterate through 3x3 squares in sudoku.

## [Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/)

Easy, just make sure to think about how can I iterate through less stuff(use a set), and make sure to iterate through the less stuff.

# Two Pointer

## [Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)

Easy, but need to be more careful with what variables I'm using. TLDR: need to do leetcode when I'm more awake

## [Two Sum II](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)

This is just binary search but not really, since we are only going one element at a time.
