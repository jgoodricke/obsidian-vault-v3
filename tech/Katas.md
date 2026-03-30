## 1. Bowling Game Kata

This is probably the most famous kata.

**Problem:**
Score a bowling game given a list of rolls.

**Skills practised:**

- TDD
- Incremental design
- Refactoring
- Handling edge cases
- Clean object modelling

This kata is excellent because the design usually starts messy and gets cleaned through refactoring.

---

## 2. Prime Factors Kata

**Problem:**
Given an integer, return its prime factors.

Example:

```
primeFactors(18) -> [2, 3, 3]
```

**Skills practised:**

- TDD cycles
- Simple algorithms
- Small functions
- Refactoring duplication
- Naming

This is one of Uncle Bob’s favourites.

---

## 3. FizzBuzz (but done properly with TDD)

This is not trivial if done correctly.

Rules:

- Multiples of 3 → Fizz
- Multiples of 5 → Buzz
- Both → FizzBuzz
- Otherwise → number

**Focus on:**

- TDD
- No duplication
- Clean naming
- Small functions
- Refactoring

You can redo this many times trying different designs:

- if statements
- lookup tables
- rule objects
- functional style
- OOP style

---

## 4. String Calculator Kata

Very popular TDD kata.

**Problem:**
Parse a string like:

```
"1,2,3" -> 6
"1\n2,3" -> 6
"//;\n1;2" -> 3
```

**Skills practised:**

- TDD
- Parsing
- Incremental complexity
- Refactoring
- Clean error handling

This is an excellent TypeScript kata.

---

## 5. Word Wrap Kata

**Problem:**
Wrap text so that each line is a maximum length.

Example:

```  
wrap("hello world", 5)
"hello\nworld"
```

**Skills practised:**

- String manipulation
- Refactoring
- Small functions
- Clean naming
- Algorithm design

---

## 6. Roman Numerals Kata

Convert numbers to Roman numerals.

```
1 -> I
4 -> IV
9 -> IX
58 -> LVIII
```

**Skills practised:**

- Mapping logic
- Refactoring
- Table-driven design
- Clean code
- Tests first

---

## 7. Bank Account Kata

Simple domain modelling kata.

Operations:

- deposit
- withdraw
- print statement
- running balance

**Skills practised:**

- Object design
- Separation of concerns
- Testing domain logic
- Clean architecture thinking

Very good for TypeScript because you can model domain objects and interfaces.

---

# How to Practise (Important)

Don’t just solve the kata once.

Do this instead:

1. Use **TDD**
2. Small steps only
3. Refactor constantly
4. Repeat the same kata many times
5. Try different designs each time
6. Try different styles:

   - functional
   - OOP
   - immutable data
   - classes vs pure functions

7. Try to get faster each time
8. Focus on **clean code**, not finishing quickly

The goal is not the solution.
The goal is **discipline and craftsmanship**.

---

# If you want a good TypeScript kata rotation

This would be a very solid rotation:

1. FizzBuzz
2. Prime Factors
3. String Calculator
4. Bowling Game
5. Roman Numerals
6. Word Wrap
7. Bank Account

Repeat forever.
