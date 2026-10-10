# kadane algorithm: How to Find the Maximum Subarray Sum in One Pass (Code, Edge Cases, and the Variants Interviewers Ask)

LeetCode 53 is the reason most people search for this. You're given an array like `[-2, 1, -3, 4, -1, 2, 1, -5, 4]`, and you need the largest sum of any contiguous chunk. The chunk here is `[4, -1, 2, 1]`, which sums to 6.

The brute-force version is easy to write and hopeless to run: check every start index, every end index, that's O(n²) subarrays, and with `n` up to 10⁵ you're looking at billions of additions. Kadane's algorithm gets the same answer in one pass with two integers. That gap — O(n²) to O(n) using constant memory — is why this shows up in interviews so often, and why "I memorized the code" tends to fall apart the moment a follow-up question lands.

## What Kadane's algorithm actually computes

The trick isn't in the loop. It's in the reframing.

Instead of asking "what is the best subarray in this whole array?", ask a smaller question for every index `i`:

> What is the largest sum of any subarray that *ends exactly at* index `i`?

Call that `f(i)`. Once you can answer that for every position, the global answer is just the biggest `f(i)` you ever saw. Nothing fancy, no backtracking.

The part that makes it linear is that `f(i)` doesn't need to be computed from scratch. Any subarray ending at `i` is either the subarray ending at `i-1` extended by one element, or just the single element `nums[i]` on its own. So:


f(i) = max(nums[i], f(i-1) + nums[i])
answer = max over all i of f(i)


Two candidate values, pick the bigger one. That's the whole algorithm.

## Tracing it on the example array

Numbers make this concrete faster than prose does. Array: `[-2, 1, -3, 4, -1, 2, 1, -5, 4]`.

| i | nums[i] | f(i) = max(nums[i], f(i-1) + nums[i]) | best so far |
| --- | --- | --- | --- |
| 0 | -2 | -2 | -2 |
| 1 | 1 | max(1, -1) = 1 | 1 |
| 2 | -3 | max(-3, -2) = -2 | 1 |
| 3 | 4 | max(4, 2) = 4 | 4 |
| 4 | -1 | max(-1, 3) = 3 | 4 |
| 5 | 2 | max(2, 5) = 5 | 5 |
| 6 | 1 | max(1, 6) = 6 | 6 |
| 7 | -5 | max(-5, 1) = 1 | 6 |
| 8 | 4 | max(4, 5) = 5 | 6 |

Watch index 1. The running sum from index 0 was -2, and extending it gives -1, which is worse than starting fresh at 1. So the subarray restarts. Same thing at index 3: the running sum was -2, extending gives 2, but 4 alone beats it.

Index 7 is the interesting row. The running sum drops from 6 all the way to 1, and yet the answer stays 6 — because `f(i)` is a local quantity, while the answer is a running maximum. Mixing those two up is the single most common implementation bug.

## The compact version you'll see in other people's code

A lot of solutions, including the walkthrough on 👉 [AlgoMonster's LeetCode 53 breakdown](https://algo.monster/liteproblems/53?fpr=li69), fold the two variables differently:

python
def maxSubArray(nums):
    ans = f = nums[0]
    for x in nums[1:]:
        f = max(f, 0) + x    # drop any negative prefix, then extend
        ans = max(ans, f)
    return ans


`max(f, 0) + x` is algebraically the same as `max(f + x, x)`: if `f` is negative, you throw it away and start at `x`. Same O(n) time, same O(1) space, just written as a reset rather than a comparison.

## Why throwing away a negative running sum is safe

This is the part that separates people who understand Kadane's algorithm from people who copied it.

Say your running sum `f(i-1)` is negative. Any subarray that contains the entire prefix ending at `i-1` has a sum of `f(i-1) + (sum of the rest)`. Since `f(i-1) < 0`, you'd get a strictly larger total by cutting that prefix off and keeping only "the rest." So no optimal subarray ever includes a negative-sum prefix — which means whenever the running sum goes negative, everything before it is dead weight.

That argument is why the greedy reading of the algorithm works. It's also why the DP reading works: `f(i)` is a proper state with optimal substructure, and you're just rolling it forward instead of storing a table.

AlgoMonster's own community forum has a thread where someone questions exactly this classification on the site's keyword-to-algorithm cheat sheet — asking whether Kadane's should be filed under dynamic programming rather than greedy. It's a fair question, and the honest answer is that it's both. The recurrence is DP. The decision rule ("negative sum → restart") is the greedy shortcut that collapses the DP table into one variable. Interviewers generally accept either label, as long as you can defend the recurrence.

## The edge case that breaks most implementations

All-negative arrays.

If you initialize your answer to `0`, then for an input like `[-3, -1, -4]` you'll return `0` — which corresponds to the empty subarray. LeetCode 53 requires at least one element, so the correct answer is `-1`. Initialize both `f` and `ans` to `nums[0]` and the problem disappears.

The same trap shows up in the reset-style code: `curSum = 0` followed by a reset count means `[-3, -1, -4]` returns `0` too. Check which convention the problem states — empty subarray allowed, or not — before picking your initialization.

Two more things worth checking in your own code:

- **Reset order.** The reset check must happen *before* adding the current element, not after. Doing it after silently changes where subarrays restart.
- **`n = 1`.** The loop body never runs, and you return `nums[0]`. That's correct, but only because you didn't seed with `0` or `-inf` in a way that leaks through.

## A historical footnote that's actually useful

The problem was proposed by Ulf Grenander at Brown University in 1977, as a way to illustrate a maximum-likelihood model for digital images. Jay Kadane, then at Carnegie Mellon, produced the linear-time solution some years later. That's the standard telling.

What's less commonly repeated: a 2023 paper in the journal *Algorithms*, authored by Kadane himself, compares two linear-time algorithms for the problem and concludes that the algorithm the world calls "Kadane's" is not the one he originally intended. Both are linear, both use a handful of words of memory, both are dynamic programming in structure — and the paper proves they differ only when the input consists entirely of negative numbers, where his intended version carries more information.

Practical upshot: if an interviewer asks what happens on an all-negative array, you have a genuinely interesting answer instead of a shrug.

## Code in the languages you'll actually be writing

Python:

python
def max_subarray(nums):
    f = best = nums[0]
    for x in nums[1:]:
        f = max(x, f + x)
        best = max(best, f)
    return best


Java:

java
public int maxSubArray(int[] nums) {
    int cur = nums[0], best = nums[0];
    for (int i = 1; i < nums.length; i++) {
        cur = Math.max(nums[i], cur + nums[i]);
        best = Math.max(best, cur);
    }
    return best;
}


Both are O(n) time and O(1) space. Neither allocates a DP array, and neither needs one — the recurrence only ever looks back one step.

## The follow-up: divide and conquer

LeetCode 53 explicitly suggests it: "If you have figured out the O(n) solution, try coding another solution using the divide and conquer approach, which is more subtle."

The idea: split the array at the midpoint. The best subarray is either entirely in the left half, entirely in the right half, or crosses the midpoint. The first two come from recursion. The third is computed in linear time by expanding outward from the middle on both sides, tracking the best left-side suffix sum and best right-side prefix sum, then adding them.

That gives you O(n log n) with O(log n) recursion depth. It's strictly worse than Kadane's on this problem — but reciting it correctly under pressure is a decent signal that you understand the structure rather than the incantation.

## Where Kadane's algorithm shows up beyond LeetCode 53

The pattern generalizes far enough that recognizing it is worth more than memorizing it. A partial list of problems that reduce to "keep the best value ending here":

- **Maximum product subarray.** Same shape, except you must also track the minimum, because a very negative number times another negative number becomes very positive.
- **Maximum circular subarray.** The answer is either a normal Kadane result, or the total sum minus the *minimum* subarray sum — with one exception when all values are negative.
- **Maximum subarray with one deletion.** Track a second state: best sum ending here *with* the deletion already used.
- **Best time to buy and sell stock.** The state is the running minimum price instead of a running sum, but the "best value so far" scan is identical.
- **Maximum sum rectangle in a 2D matrix.** Compress each pair of rows into a 1D array of column sums, then run Kadane's on it — O(n³) overall.

If a problem asks for a contiguous window whose sum, product, or difference is extremal, and the constraint rules out a sliding window, Kadane-style state tracking is usually the next thing to try.

## Practicing this without grinding 300 problems

The reason people search for Kadane's algorithm at 11pm isn't curiosity. It's that the pattern shows up in an interview in a form they don't recognize, and pattern recognition doesn't come from solving 400 unrelated problems.

That's the specific gap 👉 [AlgoMonster](https://algo.monster?fpr=li69) was built around. Instead of a flat problem bank, it's organized as a structured course: 👉 [see what's inside AlgoMonster's curriculum](https://algo.monster?fpr=li69), where the site lists 500+ lessons across 6 courses, 1,000+ illustrations, and 3,000+ editorials, plus real interview questions from tech companies that are updated monthly. The lessons are text-and-diagram based rather than video, which matters if you'd rather read a trace table like the one above than watch someone else click through it.

Worth knowing before you pay: it's created by ex-Google engineers, and it's targeted at interview prep rather than general CS education. There's no monthly plan — you're choosing between annual and lifetime — and there's also a free tier with limited access so you can see whether the teaching format fits how you read.

One more thing the free tier can't tell you: whether the problems you get after the lessons actually cover the Kadane family. The pattern-based structure is the whole pitch — a lesson on DP state compression followed by problems that require recognizing it, rather than a randomized queue.

## AlgoMonster plans and current pricing

Pricing on the site fluctuates with promotions, so check the live page rather than trusting a number in a blog post. As currently listed:

| Plan | Core access | Price | Billing | Link |
| --- | --- | --- | --- | --- |
| Free | Limited sample content — enough to judge the teaching style | $0 | None | [Start with the free tier](https://algo.monster?fpr=li69) |
| Pro – Yearly | All patterns, lessons and problems; unlimited AI assistant; unlimited Speedrun; 500+ lessons across 6 courses; 1,000+ illustrations; 3,000+ editorials; company questions updated monthly; member discount on live bootcamps | $99/year | Billed annually (listed as $8.25/mo, $0.28/day) | [Get Pro Yearly access](https://algo.monster?fpr=li69) |
| Pro – Lifetime | Everything in Yearly, plus lifetime updates | $189 | One-time payment | [Get Pro Lifetime access](https://algo.monster?fpr=li69) |

Both paid tiers also currently include bonus courses — a CS Foundation course and a "Build an LLM" course — at no extra cost. Some specialized content developed with external partners, such as the Quant Interview Prep course, is sold separately.

The break-even math is unglamorous: if you're interviewing in the next two or three months, yearly is the cheaper commitment. Lifetime only starts to make sense if you expect to be back on the interview circuit more than once. Two years of yearly equals one lifetime purchase — everything past that is the reason to buy it, and nothing before that is.

If you're not sure which pattern family your weak spot belongs to, the free tier is genuinely free, and the 👉 [Speedrun and pattern library](https://algo.monster?fpr=li69) is where the Kadane variants live.

## Quick answers to the questions people actually search

**Is Kadane's algorithm greedy or dynamic programming?**
Both readings are defensible. The state transition is DP; the "discard negative running sums" rule is a greedy simplification that collapses the DP table to a single variable.

**What's the time and space complexity?**
O(n) time — one pass, each element touched once. O(1) space — two integers, regardless of input size.

**Why does it work?**
No optimal subarray contains a negative-sum prefix, because cutting that prefix off would increase the total. So a negative running sum is always safe to discard.

**What happens with all-negative input?**
The correct answer is the largest (least negative) single element. Initialize both variables to `nums[0]`, not to `0`.

**Can it be parallelized or streamed?**
Yes to streaming — it only needs one pass, so it works on data that arrives incrementally. Parallel versions exist but are more involved, since merging two partial results requires tracking best prefix, best suffix, total sum, and best overall for each half.

The algorithm itself is nine lines. What takes practice is noticing, thirty seconds into a new problem, that it's a Kadane problem wearing different clothes.
