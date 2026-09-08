Day 19 DSA Quiz — Recursion: Fibonacci

1. What is the Fibonacci rule?

A. Each number is double the previous number  
B. Each number is the sum of the previous two numbers  
C. Each number is always prime  
D. Each number is sorted  

2. What are the base cases for Fibonacci?

A. `fib(0) = 0`, `fib(1) = 1`  
B. `fib(0) = 1`, `fib(1) = 1`  
C. `fib(1) = 0`, `fib(2) = 1`  
D. `fib(n) = n * fib(n - 1)`  

3. Correct recursive formula is:

A. `fib(n) = fib(n - 1) * fib(n - 2)`  
B. `fib(n) = fib(n - 1) + fib(n - 2)`  
C. `fib(n) = n + fib(n - 1)`  
D. `fib(n) = fib(n + 1)`  

4. What is `fib(5)` if sequence starts at index 0?

A. `3`  
B. `5`  
C. `8`  
D. `13`  

5. What is `fib(6)`?

A. `5`  
B. `6`  
C. `8`  
D. `13`  

6. Why is naive recursive Fibonacci slow?

A. It sorts the array repeatedly  
B. It repeats the same subproblems many times  
C. It uses binary search  
D. It uses too little memory  

7. Naive Fibonacci recursion usually has time complexity:

A. `O(n)`  
B. `O(log n)`  
C. `O(2^n)`  
D. `O(1)`  

8. Space complexity of naive recursive Fibonacci is usually:

A. `O(n)`  
B. `O(1)`  
C. `O(n²)`  
D. `O(2^n)`  

9. What does memoization do?

A. Sorts values  
B. Stores already calculated answers  
C. Deletes recursive calls  
D. Converts code into SQL  

10. Fibonacci with memoization has time complexity:

A. `O(2^n)`  
B. `O(n)`  
C. `O(log n)`  
D. `O(n²)`  

11. Fibonacci with memoization uses extra space mainly because of:

A. Dictionary/cache and call stack  
B. Sorting array  
C. Database storage  
D. Binary search tree  

12. Iterative Fibonacci can achieve:

A. `O(n)` time and `O(1)` space  
B. `O(2^n)` time and `O(n)` space  
C. `O(log n)` time and `O(n)` space  
D. `O(n²)` time and `O(1)` space  

13. Which is safer for memo default argument in Python?

A. `def fib(n, memo={})`  
B. `def fib(n, memo=None)`  
C. `def fib(n, memo=[])`  
D. `def fib(n, memo=set())`  

14. How many recursive calls does naive Fibonacci make in the recursive case?

A. One  
B. Two  
C. Three  
D. Zero  

15. Which is a common Fibonacci mistake?

A. Having base cases  
B. Using memoization  
C. Confusing `fib(5)` with `fib(6)`  
D. Returning cached values  

Send answers like:

```text
1.B
2.A
3.B
...
15.C
```
