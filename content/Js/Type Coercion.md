---
{"publish":true,"created":"2026-08-12T07:21:21.536Z","modified":"2026-08-17T02:09:03.154Z"}
---

#### PREMITIVES

so basically the operator decides what type coercion to perform. in case arithmetic operators other than '+' js always coerce to numbers but for '+' concatenation takes precendecen over addition. SO if any isde is string. js coerces other side also as string.

```javascript
"5" - 2        // 3  (string "5" → number 5, then 5 - 2)
"10" * "2"     // 20 (both strings → numbers, then 10 * 2)
"100" / "5"    // 20 (both strings → numbers, then 100 / 5)
"10" % "3"     // 1  (both strings → numbers, then 10 % 3)
```

When `+` sees **any string**, it chooses **concatenation** and converts everything to strings \[61]\[82]:

```javascript
5 + "2"        // "52"  (number 5 → string "5", then concatenation)
"5" + 2        // "52"  (number 2 → string "2", then concatenation)
"5" + "2"      // "52"  (both strings, concatenation)
5 + 2          // 7     (both numbers, addition)
```

**5 + "2" - 3.        Evaluated Left-to-Right (Same Precedence)**

#### NON-PREMITIVES

**Objects and arrays in math/comparison contexts (`+`, `-`, `<`, `==` with numbers):**

- Plain objects `{}` become the string `"[object Object]"`, then that gets converted further if needed
- Arrays become a comma joined string of their contents, then converted further if needed
  - `[1,2,3]` becomes `"1,2,3"`
  - `[5]` becomes `"5"`
  - `[]` becomes `""`

**Then that string may get converted to a number, if the context needs a number:**

- `"5"` becomes `5`
- `""` becomes `0`
- `"1,2,3"` becomes `NaN` (can't convert that to a number)
- `"[object Object]"` becomes `NaN`

```
[5] + 1        // "5" + 1 → "51" (string concat wins if either side is a string)
[5] - 1        // "5" - 1 → 5 - 1 → 4 (minus forces numeric)
[5] == 5       // "5" == 5 → 5 == 5 → true
[] == false    // "" == false → 0 == 0 → true
{} + 1         // "[object Object]" + 1 → "[object Object]1"
{} - 1         // NaN
```

**Simple rules to remember:**

1. Arrays turn into their contents joined by commas (empty array becomes empty string).
2. Plain objects turn into `"[object Object]"` (mostly useless, basically always breaks math).
3. `+` prefers strings (if anything nearby is a string like, concatenate).
4. `-`, `*`, `/`, `<`, `>` prefer numbers (force numeric conversion).
5. Empty array `[]` is sneaky, it becomes `""` then `0`, so it often acts like zero or falsy in comparisons.
6. Dates are the odd one out, they lean toward becoming strings by default, not numbers.

#### DATE

So date around a + operator behaves as string and hence concetanation.

And for numeric operation date automatically behaves as epoch timestamp which is a number

```
const d = new Date();

console.log(d + 1);
// "Wed Aug 12 2026 00:00:00 GMT+0530 (India Standard Time)1"
// Date leans toward string by default, so + triggers string concat

console.log(d - 0);
// 1755000000000 (some timestamp number)
// minus forces numeric conversion, gives you the epoch timestamp

console.log(d * 1);
// same as above, timestamp number
// multiplication also forces numeric

console.log(+d);
// timestamp number
// unary plus also forces numeric, common trick to get epoch ms

console.log(`${d}`);
// "Wed Aug 12 2026 00:00:00 GMT+0530 (India Standard Time)"
// template literals force string

console.log(d == d.toString());
// true
// comparing to a string uses Date's string form

console.log(d > 0);
// true
// comparison operators force numeric, so this compares timestamp to 0

console.log(new Date(2026, 0, 1) < new Date(2027, 0, 1));
// true
// both dates coerced to numbers (timestamps) for comparison
```
