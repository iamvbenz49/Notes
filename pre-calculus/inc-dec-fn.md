# Increasing and Decreasing Functions

## Definitions

### Increasing Function
A function $f$ is **increasing** on an interval $I$ if:

$$x_1 < x_2 \implies f(x_1) < f(x_2) \quad \forall\ x_1, x_2 \in I$$

> The graph **climbs from left to right** on that interval.

### Decreasing Function
A function $f$ is **decreasing** on an interval $I$ if:

$$x_1 < x_2 \implies f(x_1) > f(x_2) \quad \forall\ x_1, x_2 \in I$$

> The graph **falls from left to right** on that interval.

### Strictly vs. Non-Strictly
| Type | Condition |
|------|-----------|
| Strictly Increasing | $x_1 < x_2 \Rightarrow f(x_1) < f(x_2)$ |
| Non-decreasing (Weakly Increasing) | $x_1 < x_2 \Rightarrow f(x_1) \leq f(x_2)$ |
| Strictly Decreasing | $x_1 < x_2 \Rightarrow f(x_1) > f(x_2)$ |
| Non-increasing (Weakly Decreasing) | $x_1 < x_2 \Rightarrow f(x_1) \geq f(x_2)$ |

---

## Key Properties

### 1. Derivative Test (for differentiable functions)
If $f$ is differentiable on $(a, b)$:

- $f'(x) > 0 \ \forall\ x \in (a,b)$ → $f$ is **strictly increasing** on $(a, b)$
- $f'(x) < 0 \ \forall\ x \in (a,b)$ → $f$ is **strictly decreasing** on $(a, b)$
- $f'(x) = 0 \ \forall\ x \in (a,b)$ → $f$ is **constant** on $(a, b)$

> ⚠️ $f'(x) \geq 0$ (with equality only at isolated points) still guarantees strictly increasing.

### 2. Monotonicity
A function that is either entirely increasing or entirely decreasing on an interval is called **monotonic** on that interval.

### 3. Inverse Functions
- If $f$ is **strictly monotonic** on $I$, then $f$ is **one-to-one (injective)** on $I$, and its inverse $f^{-1}$ exists.
- If $f$ is strictly increasing → $f^{-1}$ is also strictly increasing.
- If $f$ is strictly decreasing → $f^{-1}$ is also strictly decreasing.

### 4. Composition Rules
| $f$ | $g$ | $f \circ g$ |
|-----|-----|-------------|
| Increasing | Increasing | Increasing |
| Decreasing | Decreasing | Increasing |
| Increasing | Decreasing | Decreasing |
| Decreasing | Increasing | Decreasing |

### 5. Algebra of Monotonic Functions
- Sum of two increasing functions → **increasing**
- Sum of two decreasing functions → **decreasing**
- Product: depends on signs (not guaranteed to preserve monotonicity)

---

## Finding Intervals of Increase/Decrease

**Step-by-step method:**

1. Find $f'(x)$
2. Solve $f'(x) = 0$ and find where $f'(x)$ is undefined → these are **critical points**
3. Use critical points to divide the domain into sub-intervals
4. Pick a test value in each sub-interval and evaluate the sign of $f'(x)$
5. - $f'(x) > 0$ → increasing on that interval  
   - $f'(x) < 0$ → decreasing on that interval

---

## Example

$$f(x) = x^3 - 3x$$

$$f'(x) = 3x^2 - 3 = 3(x-1)(x+1)$$

Critical points: $x = -1,\ x = 1$

| Interval | Test value | $f'(x)$ sign | Behaviour |
|----------|------------|--------------|-----------|
| $(-\infty, -1)$ | $x = -2$ | $+$ | Increasing ↑ |
| $(-1, 1)$ | $x = 0$ | $-$ | Decreasing ↓ |
| $(1, \infty)$ | $x = 2$ | $+$ | Increasing ↑ |

---

## Quick Reference

```
Increasing:  x1 < x2  →  f(x1) < f(x2)   [graph goes up L→R]
Decreasing:  x1 < x2  →  f(x1) > f(x2)   [graph goes down L→R]
f'(x) > 0  →  Increasing
f'(x) < 0  →  Decreasing
f'(x) = 0  →  Critical point (potential max/min or inflection)
```
