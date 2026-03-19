# Functional vs Non-Functional Graphs — Notes

---

## Core Idea

A graph is **functional** if every input (x) has **exactly one output (y)**.

A graph is **non-functional** if any input (x) has **more than one output (y)**.

---

## The Vertical Line Test

The quickest way to tell — draw (or imagine) a **vertical line** anywhere on the graph.

- Vertical line touches the graph **once** → **functional**
- Vertical line touches the graph **more than once** → **non-functional**

> **A function can never have two points directly above each other on a graph.**

---

## Functional Graphs ✅

These pass the vertical line test.

### Straight Line
```
y = 2x + 1
```
Every x gives exactly one y.

### Parabola (opening up or down)
```
y = x²
```
x = 2 gives only y = 4.

### Square Root
```
y = √x
```
Only takes the positive root — one output per input.

### Absolute Value
```
y = |x|
```
Looks like it might have two outputs but doesn't — at x = 2, y = 2 only.

### Horizontal Line
```
y = 5
```
Every x maps to the same single value. Functional but **not one-to-one** — many inputs share the same output.

---

## Non-Functional Graphs ❌

These fail the vertical line test.

### Circle
```
x² + y² = 9
```
At x = 0, y = +3 **and** y = -3. Two outputs for one input.

### Sideways Parabola
```
x = y²
```
At x = 4, y = +2 **and** y = -2.

### Vertical Line
```
x = 3
```
x = 3 maps to **every** y value — infinite outputs for one input.

---

## Summary Table

| Graph | Functional? | Why |
|---|---|---|
| Straight line | ✅ | One y per x |
| Parabola (up/down) | ✅ | One y per x |
| Square root | ✅ | One y per x (positive only) |
| Absolute value | ✅ | One y per x |
| Horizontal line | ✅ | One y per x |
| Circle | ❌ | Two y values for middle x values |
| Sideways parabola | ❌ | Two y values for positive x |
| Vertical line | ❌ | Infinite y values for one x |
