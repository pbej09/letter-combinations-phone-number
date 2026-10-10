# Letter Combinations of a Phone Number: Backtracking Walkthrough With Python, Java, and C++ Code You Can Defend in an Interview

The digit-to-letter mapping is not where people lose this problem. Everyone can write down `2 → abc`. What trips candidates up is the recursion: keeping the current string in the right state, knowing where to stop, and being able to say out loud why the loop never needs a start index. Then the interviewer asks what happens when the input is a hundred digits long, and the room gets quiet.

This is LeetCode 17, rated Medium, tagged Hash Table / String / Backtracking. It shows up constantly in phone screens because it tests one specific skill: building a solution incrementally over a decision tree and cleaning up after yourself. Here's the full walkthrough — approach, code in three languages, complexity, the bugs that actually cost offers, and the follow-ups that separate "I memorized it" from "I understand it."

## What the problem actually asks

You get a string of digits from `2` through `9`. Return every letter combination those digits could represent, using the standard telephone keypad layout. The digit `1` maps to nothing.

| Digit | Letters |
| --- | --- |
| 2 | abc |
| 3 | def |
| 4 | ghi |
| 5 | jkl |
| 6 | mno |
| 7 | pqrs |
| 8 | tuv |
| 9 | wxyz |

Two things in that table matter more than they look. Digits `7` and `9` carry **four** letters while the rest carry three — that asymmetry is the entire source of the ugly-looking complexity bound you'll need to recite. And `1`, `0`, `*`, `#` map to nothing, which is where input-validation questions come from.

Examples:

- `digits = "23"` → `["ad","ae","af","bd","be","bf","cd","ce","cf"]`
- `digits = "2"` → `["a","b","c"]`
- `digits = ""` → `[]`

LeetCode constrains the input to a length between 0 and 4, and answers may be returned in any order. In an interview, say both of those out loud before you code — the length limit tells you exponential time is fine, and the "any order" clause tells you not to waste effort sorting.

## Why this is a backtracking problem

Draw it. The root is the empty string. Level one branches into the letters of the first digit. Level two branches from each of those into the letters of the second digit. Every root-to-leaf path is one answer.


        ""
     /   |   \
    a    b    c
   /|\  /|\  /|\
 ad ae af bd be bf cd ce cf


Depth equals `len(digits)`. Branch count at each level equals the letter group size. That shape is a decision tree, and "enumerate every path through a decision tree" is the definition of backtracking.

The detail that separates a clean solution from a mangled one: the loop over letters starts at the **beginning of the current digit's group**, not at some `startIndex`. In problems like Combination Sum or Subsets you carry a `startIndex` forward so you don't reuse elements from the same set. Here you're combining across *different* sets — the letters for digit 1 and the letters for digit 2 are independent groups — so there is nothing to skip. Copy a template with `startIndex` baked in and you'll either skip valid combinations or produce duplicates.

## The backtracking solution, step by step

1. If `digits` is empty, return `[]` immediately. This guard is required, not optional.
2. Build the digit-to-letters lookup. A dictionary keyed by character works; an array indexed by `digit - 2` is faster and shorter.
3. Maintain a `path` holding the letters chosen so far for the current branch.
4. Recurse on an index `i` representing how many digits you've consumed.
5. Base case: when `i == len(digits)`, the path is a complete answer. Copy it into the result and return.
6. Otherwise loop over `mapping[digits[i]]`: append a letter, recurse with `i + 1`, then remove the letter you appended.

That last step — the removal — is the "backtrack" in backtracking. It restores the state so the next letter at this level starts from a clean path.

Python:

python
class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        if not digits:
            return []

        mapping = {
            "2": "abc", "3": "def", "4": "ghi", "5": "jkl",
            "6": "mno", "7": "pqrs", "8": "tuv", "9": "wxyz",
        }
        res: List[str] = []
        path: List[str] = []

        def backtrack(i: int) -> None:
            if i == len(digits):
                res.append("".join(path))
                return
            for ch in mapping[digits[i]]:
                path.append(ch)
                backtrack(i + 1)
                path.pop()

        backtrack(0)
        return res


Tracing `"23"`: `i=0` picks `a`, `i=1` picks `d` → `ad` is recorded, `d` is popped, `e` is pushed → `ae`, then `af`, then `a` itself is popped and the same cycle runs from `b`. Nine leaves, nine answers.

Using a list for `path` and joining once at the leaf keeps the cost proportional to the answer length. Building immutable strings with `path + ch` at every call is what interviewers flag as sloppy in Python, even though it produces the same answers.

## The same solution in Java and C++

Java:

java
class Solution {
    private static final String[] MAP = {
        "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"
    };

    public List<String> letterCombinations(String digits) {
        List<String> res = new ArrayList<>();
        if (digits.isEmpty()) return res;
        backtrack(digits, 0, new StringBuilder(), res);
        return res;
    }

    private void backtrack(String digits, int i, StringBuilder path, List<String> res) {
        if (i == digits.length()) {
            res.add(path.toString());
            return;
        }
        for (char ch : MAP[digits.charAt(i) - '2'].toCharArray()) {
            path.append(ch);
            backtrack(digits, i + 1, path, res);
            path.deleteCharAt(path.length() - 1);
        }
    }
}


C++:

cpp
class Solution {
    const string MAP[8] = {"abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
public:
    vector<string> letterCombinations(string digits) {
        vector<string> res;
        if (digits.empty()) return res;
        string path;
        dfs(digits, 0, path, res);
        return res;
    }

    void dfs(const string& digits, int i, string& path, vector<string>& res) {
        if (i == (int)digits.size()) {
            res.push_back(path);
            return;
        }
        for (char ch : MAP[digits[i] - '2']) {
            path.push_back(ch);
            dfs(digits, i + 1, path, res);
            path.pop_back();
        }
    }
};


The `-'2'` offset is worth pausing on. Because the array starts at digit `2`, the correct index is `digit - 2`. Writing `digit - '0'` gives you `2` for input `"2"`, which overruns the first slot, and writing `digit - '1'` shifts every digit by one so `"2"` returns `def`. Both are extremely common twenty-minute mistakes.

## Complexity: what to say out loud

Let `n` be the number of digits, with `m` of them mapping to three letters and `k` mapping to four (`m + k = n`). The number of answers is at most `3^m · 4^k`, which people usually write as `O(4^n)` for simplicity. Constructing each answer costs `O(n)`, because the path has length `n` every time.

- **Time:** `O(3^m · 4^k · n)`, or `O(4^n · n)` worst case.
- **Space:** `O(n)` for the recursion stack and the path, plus `O(4^n · n)` for the output you're required to return.

The line that earns points: the output itself is exponential, so *every* correct solution is exponential. The only room for improvement is not doing work beyond copying each answer, and this solution doesn't.

## The iterative version (worth knowing, not always worth leading with)

You can build the answers level by level instead of recursing. Start with `[""]`. For each digit, replace every partial string with itself plus each letter of that digit's group.

python
class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        if not digits:
            return []
        groups = ["abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"]
        res = [""]
        for digit in digits:
            res = [prefix + ch for prefix in res for ch in groups[int(digit) - 2]]
        return res


Same time and space bounds, no recursion depth, and the code fits on one line of the loop. It's the version to pull out when an interviewer asks what happens if the input string gets long — Python's default recursion limit is 1000 frames, and while you'll never hit that at LeetCode's length cap, real phone numbers with extensions or a deliberately hostile interviewer can push you there. It also naturally produces lexicographic order when the letter groups are stored in alphabetical order, which answers the "can you return them sorted?" follow-up for free.

## Bugs that actually cost interviews

- **Off-by-one in the mapping.** Covered above. Test with `"2"` and `"9"`, not just `"23"` — that's how you catch it in the room.
- **Returning `[""]` for empty input.** The base case `res = [""]` combined with a missing empty guard produces exactly one wrong answer, on exactly one test case. LeetCode will catch it; a distracted interviewer might not, and then you ship the bug.
- **Mutating the result list while iterating it.** `result.append(result[i] + ch)` inside a loop over `result` runs forever. Build a separate list each pass.
- **Forgetting the pop.** Skip the cleanup and every subsequent branch inherits leftover characters. The first wrong output looks like garbage rather than an obvious mistake, so it burns time.
- **Java-specific: appending the `StringBuilder` itself.** You'll store references to one object and end up with `n` identical strings. Call `.toString()`.
- **Unvalidated input.** If the interviewer says the input might include `0`, `1`, `*`, or letters, decide out loud whether to skip them, raise an error, or treat them as empty groups. There's no single right answer; picking one deliberately is the answer.

## Follow-ups interviewers actually ask

**"What if the string is 100 digits long?"** The output is astronomically large, so the honest answer is that you'd need to question whether the caller really wants all of it. If they do, use the iterative version to avoid stack limits, and consider a generator so you're not holding `4^100` strings in memory.

**"Can you produce results lazily?"** Yes. Yield at each leaf instead of appending, or turn the iterative version into a generator over partial strings. This is the natural answer to memory pressure.

**"What if two digits map to overlapping letter sets?"** Nothing changes — you're indexing by position, not by letter content, so duplicates in the mapping don't produce duplicates in the output.

**"Why is this Medium?"** Because the recursion is easy to write wrong and the follow-ups reward understanding. Nobody fails this problem on the mapping.

**"Can you do it without any recursion?"** The iterative product version above. Say that out loud even if you wrote the recursive one — it costs ten seconds and demonstrates you can see the alternative.

## Why this problem is worth more than one solution

Here's the pattern-level issue. LeetCode 17 is the friendliest member of a family that includes Subsets, Permutations, Combination Sum, Palindrome Partitioning, Word Search, and N-Queens. They all reduce to the same three moves: choose, recurse, un-choose. Learn LeetCode 17 as a phone-keypad trick and you'll re-derive the same recursion five more times, once per problem, under interview clock pressure. Learn it as the backtracking template and the other five become variations on a shape you already know.

That reframing is the premise behind AlgoMonster, built by ex-Google engineers and structured around named patterns rather than a problem pile. Its backtracking material gives you the choose/recurse/un-choose template with a decision flowchart for spotting when a new problem fits it, then drills it across related problems so the recognition is automatic instead of reconstructed. The platform also publishes a free editorial for this exact problem, which is a fair way to see how it teaches before spending anything.

A few specifics, so you know what you're evaluating:

- **48 named patterns**, with lessons, code templates, and practice problems mapped to each one
- **3,000+ editorials** and thousands of interactive diagrams, with content written in six programming languages
- **Speedrun mode** for timed review, plus an AI assistant for hints when you're stuck
- **Text and diagram based** — there are no video walkthroughs, which is a real trade-off if you learn by watching someone code
- **Bonus courses** included in the subscription: a CS foundation course and a "build an LLM from scratch" course
- **A free tier**, so you can test the teaching style before paying

Pricing is the one place to be careful, because AlgoMonster runs sales aggressively and the numbers move. Different captures of its subscribe page have shown the annual plan at **$69.99/year**, **$99/year**, and **$119/year**, and lifetime at **$89**, **$189**, and higher — the deepest discounts land during holiday sales. There is no monthly option, and there is no formal free trial beyond the free tier.

| Plan | What you get | Price observed | Billing |
| --- | --- | --- | --- |
| Free | Limited subset of patterns and problems | $0 | — |
| Pro Yearly | All patterns, lessons and problems; unlimited AI assistant and Speedrun; company question sets; bonus Foundation and LLM courses | $69.99–$119/year depending on the active sale | Billed annually |
| Pro Lifetime | Everything in the annual plan, plus lifetime content updates | $89–$189 one-time depending on the active sale | One-time |

👉 [Check AlgoMonster's current plans and today's price](https://algo.monster?fpr=li69)

Two honest notes on value. If you're interviewing in the next six weeks, the annual plan covers your whole cycle and there's no reason to buy lifetime. If you expect to change jobs every couple of years — the normal pattern in this industry — lifetime pays for itself after roughly two cycles. And if you learn best from video, this platform will frustrate you; that's a stated design choice, not a gap they're hiding.

Also worth saying plainly: no course teaches this problem's recursion *for* you. You still have to type the code and trace `"23"` on paper until the pop step feels obvious. What a structured pattern course buys you is not skipping that work — it's doing it once instead of six times.

## FAQ

**Is `O(4^n)` time acceptable for this problem?**
Yes. The output has up to `4^n` entries, so you cannot do better than exponential. LeetCode caps input length at 4 partly because of this.

**Does the order of my results matter?**
No. The problem accepts any order. Test cases compare sets, not sequences.

**Should I use a dictionary or an array for the mapping?**
Either. The array indexed by `digit - 2` is shorter and slightly faster; a dictionary reads more clearly to reviewers who don't already know the keypad layout. Pick one and mention the alternative.

**Can I solve it with BFS and a queue?**
Yes, and it's the same algorithm — the iterative version above *is* a level-by-level traversal. Queue-based code is just a wordier way to write the same product expansion.

**Do I need to handle `1`, `0`, or `*`?**
Not for LeetCode's test data, since the constraints are `2`–`9`. In a live interview, ask. Handling unexpected characters deliberately is a cheap way to look careful.

If you want the pattern rather than one more memorized answer,👉 [get the AlgoMonster backtracking framework and run it against the full problem set](https://algo.monster?fpr=li69) — the free tier is enough to judge whether the text-and-diagram format clicks for you before you pay anything.
