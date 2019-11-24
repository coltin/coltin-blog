+++
draft = false
date = 2019-11-20T20:15:22-05:00
title = "Common Child Analysis"
description = "A breakdown of Hacker Ranks Common Child problem"
slug = ""
tags = []
categories = []
externalLink = ""
series = []
+++

Today we're going to look a problem on Hacker Rank called ["Common Child"](https://www.hackerrank.com/challenges/common-child/problem). It is ranked as a "Medium" problem, and while the problem is pretty straightforward to brute force, getting it to run without stack overflowing was fairly interesting. First let's describe the problem, brute force it, brutish force it, and then see if we can do better.

## The Problem

We are given two strings of equal length and asked to find the length of the longest common child. The common child is defined as deleting 0 or more characters from a string. Deleting characters in this way makes them a subsequence. So given the string "1234" all the possible common children (subsequences) are `{"", "1", "2", "3", "4", "12", "13", "14", "23", "24", "34", "123", "124", "134", "234", "1234"}`, of which there are 16. In general there will be <code>2<sup>{length(string)}</sup></code> of them.

As an aside, why 2<sup>n</sup> where n is the length of the string? One interesting way to think about it is when you're generating the subsequences you make a yes or a no decision for each character and you do this n times. So if you look at the list above, you will see half of them will have a 1, and half of them will not. Go ahead and count, I'll wait. So you can reduce any subsequence count by taking out a character but multiplying the answer by 2. Let's try it! `subseq("1234")` = `2 * subseq("234")` = `2*2*subseq("34")` = `2*2*2*subseq("4")` = `2*2*2*2` = <code>2<sup>4</sup></code> = `16`.

If the deconstruction is hard to think about, do it in reverse. You have `subseq("1")` is just the empty set, and the set with "1" in it. To get `subseq("12")` you take those two sets and duplicate them, adding a 2 to half of them. So `{"", "", "1", "1"}` without the 2 and then `{"", "2", "1", "12"}` with the 2 on half of them. You get the same process for adding a `"3"`, and you'll end up doubling each time.

## Brute Force!

To solve this problem let's look at the brute force first. Given strings A="qqz1234zz" and B="a1b2cnam4" the longest common child is "124". The brute force way of doing this is to look at every common subsequence of A, and check if it matches every subsequence of B. This is a lot of comparisons though. For each 2<sup>9</sup> subsequences in A, you need to look at 2<sup>9</sup> subsequences in B to see if they match. You can reduce this a bunch if you only look at subsequences of the appropriate lengths, but still, for any sufficiently large string this is pretty hairy. Ours is roughly 250k comparisons, but for strings of length 15 we're already up to a billion.

## Less brutish.

My first approach to this problem was to traverse both strings from start to end and decide how to increment the pointers based on the current character. We will call our index pointers A<sub>i</sub> and B<sub>i</sub>. As we travel through the string we have two cases:

1. The character at each location matches. If this is the case the answer will be <code>1 + solution(A<sub>i+1</sub>, B<sub>i+1</sub>)</code>. This means we chop off the string at the current location.

2. The characters do not match, in which case we have two choices. Either the longest common child is at (A<sub>i+1</sub>, B<sub>i</sub>) or at (A<sub>i</sub>, B<sub>i+1</sub>). We don't care about (A<sub>i+1</sub>, B<sub>i+1</sub>) because that will be covered by this recusrive process (possibly by either or both branches we just made). So our answer when the characters don't match is <code>Math.max(solution(A<sub>i+1</sub>, B<sub>1</sub>), solution(A<sub>i</sub>, B<sub>i+1</sub>))</code>.

This is essentially the brute force framed as a recursive algorithm. For strings that are the same, like A="aaa" and B="aaa" the function will only be called 3 times, it's just a straight line to the answer. But when characters don't match, we need to look down each possible branch. Discard a letter in A and keep B, or keep A and discard a letter in B.

This grows very fast, but the trick to making it fast is to realize that can memoize the sub-solutions. When you know the answer at (x, y) you record it for future iterations. For example, when characters don't match you have a breakdown like:

```
Step 1: sol(0, 0)
Step 2: = max(sol(1, 0),                 sol(0, 1))
Step 3: = max(max(sol(2, 0), sol(1, 1)), max(sol(1, 1), sol(0, 2)))
```

Step 1 doesn't match so it breaks down in the max of the two possible solutions in step 2. Here both `sol(1, 0)` and `sol(0, 1)` don't have a matching character so they each split and already we can notice they both want to check `sol(1, 1)`. If you trace the depth down further you get an incredible amount of overlapping questions. So many sub calls will look for `sol(4, 4)` or `sol(2, 8)`. So when you memoize the answer, you save a lot of calculations and recursive depth. So what does a solution like this look like in Java?

{{< highlight go "linenos=table" >}}

public class CommonChild {
  private int[][] memoize;
  private char[] s1;
  private char[] s2;

  public int solve(String s1, String s2) {
    memoize = new int[s1.length()][s2.length()];
    this.s1 = s1.toCharArray();
    this.s2 = s2.toCharArray();
    for (int i = 0; i < s1.length(); i++) {
      for (int j = 0; j < s2.length(); j++) {
        memoize[i][j] = -1;
      }
    }
    return moveAndFind(0, 0);
  }

  /**
   * This works and is very fast, but the primary issue is that it will stack overflow when the
   * strings get too large.
   */
  private int moveAndFind(int s1i, int s2i) {
    if (s1i >= s1.length || s2i >= s2.length) {
      return 0;
    } else if (memoize[s1i][s2i] != -1) {
      return memoize[s1i][s2i];
    }

    if (s1[s1i] == s2[s2i]) {
      memoize[s1i][s2i] = 1 + moveAndFind(s1i + 1, s2i + 1);
    } else {
      memoize[s1i][s2i] = Math.max(moveAndFind(s1i + 1, s2i), moveAndFind(s1i, s2i + 1));
    }

    return memoize[s1i][s2i];
  }
}


{{< / highlight >}}

Note: You would test this with something like `assertEquals(3, new CommonChild().solve("qqz1234zz", "a1b2cnam4"));`

You can ignore almost everything in the `solve()` method, it takes the two strings, initializes/resets the memoize array which stores subproblems, and then makes a call to `moveAndFind(0, 0)`, which will eventually return the answer. Converting to `char[]` is in no way necessary, but I like to like at the array for readability, instead of calling `s1.charAt(sl1)` for example.

All of the logic comes from `moveAndFind()`. This takes the two pointers for where you are on each of the strings. If our indicies extend past either string, we return 0 as an answer, which is our termination. Before we try to calculate the answer, we check memoize for whether the answer has already been calculated. If not, we have our two cases.

In the first case if the characters match, we calculate our answer `1 + moveAndFind(s1i + 1, s2i + 1)` and assign it to `memoize[s2i][s2i]`, and then return it.

When the characters don't match our formula is `Math.max(moveAndFind(s1i + 1, s2i), moveAndFind(s1i, s2i + 1))` which results in the branching calls.

This solution is incredibly fast and if you run most test cases, it will work no problem. The issue with this solution is that it violates an important consideration; this has a huge call stack. To calculate (0, 0) you need to calculate EVERYTHING. To calculate (0, 1) you need to calculate almost everything. And so on. So you end up with a huge call stack, and with any sufficiently large strings you are going to get a Stack Overflow.

Le poop. But we can do better.

*Exercise for the reader*: How many calculations does adding memoize save? Can you work out the math given lengths for A and B? How would you change the solution to output this answer? Does it depend on the length of the solution at all? How about the number of matching characters?

## Sweet Iterative Salvation

So our solution is fast, but the stackoverflow is an issue and just wasteful. Memoizing is a good use of space, but the call stacks are not, they are just lazy and we can do better. What we need is a non-recursive way to traverse the strings. We could unwrap the recursive function by using a stack to mimic the machine code (basically implementing recursion) but it's still wasting space, and just skirting around the JVM weakness (Stackoverflow). It might be a fun exercise to implement this, but for now we can do better.

The problem with the recursive solution is that to calculate `sol(0, 0)` we need to know either `sol(1, 1)` or `sol(0, 1) and sol(1, 0)`, and then each of those needs to know even more pieces. Once you get to the end of the line, you start collapsing the solutions and resolving the plus ones and the `Math.max()`s. So theres our secret; start with that and work backwards.

As an interesting sidenote, the direction doesn't actually matter. You could do this whole thing from the beginning and move forward, but I find it easier to think in this direction.

Perhaps too much in the weeds, but we are also going to clean up our solution by expanding the memoize by 1 in both directions and initializing the ends to 0. This saves us from having to check during the main logic, making it easier to read and debug. Here is what it looked like in our previous solution

{{< highlight go "linenos=table" >}}
    if (s1i >= s1.length || s2i >= s2.length) {
      return 0;
    } else if (memoize[s1i][s2i] != -1) {
      return memoize[s1i][s2i];
    }
{{< / highlight >}}

That `if > length` logic is not necessary if we use a bit more ram. In Java we would be fine to not initializing the ends to 0, ints are by default 0, but I like to be explicit for anyone reading the code.

Back to the solution, we're basically going to fill up the memoize from the end and work our way backwards. So the first thing we'll look at is the last characters of each string. If they match, then the answer at `memoize[end][end]` will be 1, and 0 otherwise. When we want to calculate `(end, end-1)` (which is next, and then `(end, end-2))` we have our two scenarios. If the characters match then `memoize[end][end-1] = 1 + memoize[end+1][end]` and if they don't match it will be `Math.max(memoize[end+1][end-1], memoize[end][end])`. We know the answers to all these, great, and we can continue down the line to `(end, end-2)` and so on. Once we calculate `(end, 0)` we are ready to move on to `(end-1, end)` and then `(end-1, end-1)`. I feel like I'm over-explaining this, but hopefully you're convinced that in this manner for any `(x, y)` in this ordering we will know the answer to the possible 3 questions: `(x+1, y+1)`, `(x+1, y)`, and `(x, y+1)`.

And we end up with this masterpiece:

{{< highlight go "linenos=table" >}}

public class CommonChild {
  private int[][] memoize;
  private char[] s1;
  private char[] s2;

  public int solve(String s1, String s2) {
    memoize = new int[s1.length() + 1][s2.length() + 1];
    this.s1 = s1.toCharArray();
    this.s2 = s2.toCharArray();
    for (int i = 0; i < s1.length(); i++) {
      for (int j = 0; j < s2.length(); j++) {
        memoize[i][j] = -1;
      }
    }
    for (int i = 0; i < s1.length(); i++) {
      memoize[i][s2.length()] = 0;
    }
    for (int j = 0; j < s2.length(); j++) {
      memoize[s1.length()][j] = 0;
    }

    return findLcsIterative();
  }

  private int findLcsIterative() {
    for (int i = s1.length - 1; i >= 0; i--) {
      for (int j = s2.length - 1; j >= 0; j--) {
        if (s1[i] == s2[j]) {
          memoize[i][j] = memoize[i + 1][j + 1] + 1;
        } else {
          memoize[i][j] = Math.max(memoize[i + 1][j], memoize[i][j + 1]);
        }
      }
    }
    return memoize[0][0];
  }
}

{{< / highlight >}}

Ahhh there we go. Fast. Iterative. No stack overflows. It's elegant, I love it. This is literally just least common subsequence problem, and a similar idea can be applied to many problems.

The most interesting part of this problem was definitely the conversion from the recursive problem to the iterative, and ending up not with a more convoluted solution, but something a little bit simpler, and much more efficient. I'm still not a huge fan of the memory use, but I feel like we don't need an `O(n*m)` space matrix for memoize. I think we can probably use a linear amount of space, but I think we'll leave this where it is for now.

Another interesting question to ask is, how would you return the actual longest common child? Given the `memoize` grid we could probably use that to get back the subsequence, which would be unlikely in a potentially linear solution unless you're storing the characters, which is also possible. Lots of interesting things to consider!
