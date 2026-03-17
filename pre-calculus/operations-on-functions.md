# Operations on Functions & Domain Restrictions — Notes

---

## Core Idea: Domain Baggage

When you combine two functions through any operation, the result **inherits the restrictions of both original functions** — permanently.

These restrictions are called **"domain baggage"** (or "skeletons in the closet"). They never go away, even if algebra makes them disappear.

---

## Why Restrictions Are Permanent

A function is just a **machine** — you put x in, you get a value out.

When you do `f/g`, you're not creating a new independent machine. You're **chaining** the original machines:

> "First run x through f, first run x through g, then divide the outputs."

So if x = some value makes **either machine break**, the whole chain breaks — no matter how clean the final simplified form looks.

### The Cancellation is Misleading

When a term like `(3x - 2)` cancels out during simplification, the algebra is saying:

> "These two things are equal... **as long as x ≠ 2/3**"

The cancellation itself **assumes** the restriction to be valid. So the restriction was baked into the very step that made it disappear.

---

## Steps for Operations on Functions

1. **Find individual domains first** — before doing any algebra, find restrictions for each function separately.
   - Denominator = 0 → restricted
   - Radicand (inside square root) < 0 → restricted

2. **Lock in the baggage** — these restrictions carry forward to the final answer, no matter what.

3. **Perform the operation** — add, subtract, multiply, or divide.
   - For division: multiply the first function by the reciprocal of the second.

4. **Find new restrictions** — the simplified result may introduce *additional* restrictions (e.g., a new denominator).

5. **Combine all restrictions** — original baggage + any new restrictions = final domain.

---

## Example 1: Rational Functions

$$f(x) = \frac{2x+3}{3x-2}, \quad g(x) = \frac{4x}{3x-2}$$

**Step 1 — Original domain:** Set `3x - 2 = 0` → **x ≠ 2/3** (applies to all operations)

### Addition
$$f + g = \frac{6x+3}{3x-2}$$
Domain: **x ≠ 2/3**

### Subtraction
$$f - g = \frac{-2x+3}{3x-2}$$
Domain: **x ≠ 2/3**

### Multiplication
$$f \times g = \frac{8x^2+12x}{(3x-2)^2}$$
Domain: **x ≠ 2/3**

> Leave the denominator factored — makes it easier to spot vertical asymptotes when graphing.

### Division
$$\frac{f}{g} = \frac{2x+3}{3x-2} \times \frac{3x-2}{4x} = \frac{2x+3}{4x}$$

- `(3x - 2)` cancels out — but **x ≠ 2/3 still applies** (it existed in both original functions)
- New denominator `4x` introduces a **new restriction: x ≠ 0**

Domain: **x ≠ 2/3 and x ≠ 0**

---

## Example 2: Square Root + Rational Function

$$f(x) = \sqrt{x+3}, \quad g(x) = \frac{5}{x}$$

**Step 1 — Original domains:**
- `f(x)`: radicand must be ≥ 0 → `x + 3 ≥ 0` → **x ≥ -3**
- `g(x)`: denominator can't be 0 → **x ≠ 0**

Combined original domain: **x ≥ -3 and x ≠ 0**

### Division
$$\frac{f}{g} = \sqrt{x+3} \div \frac{5}{x} = \frac{x\sqrt{x+3}}{5}$$

Looking at the result alone, `x` is no longer in the denominator — so x = 0 seems fine.

**But x ≠ 0 still applies**, because `g(x) = 5/x` was undefined there. You'd be dividing by something that doesn't exist.

Domain: **x ≥ -3 and x ≠ 0**

---

## Context Matters: Standalone vs. Built Expression

| Situation | Do you add historical restrictions? |
|---|---|
| Given only the final simplified expression | ❌ No — only restrict what *it* breaks |
| Given the original f and g, then combined | ✅ Yes — carry forward all original baggage |

### Example

$$h(x) = \frac{2x+3}{4x}$$

- Given standalone → Domain: **x ≠ 0** only
- Given as `f(x)/g(x)` where both had `x ≠ 2/3` → Domain: **x ≠ 0 and x ≠ 2/3**

The restriction x ≠ 2/3 is **not a property of the expression** — it's a property of **how it was built**.

---

## Intuition Summary

> You can't divide two things that don't exist.

If `f(2/3)` = undefined and `g(2/3)` = undefined, then `f(2/3) / g(2/3)` is meaningless — not because the division is hard, but because **there's nothing to divide in the first place**. The simplified expression just doesn't know this history. But you do.
