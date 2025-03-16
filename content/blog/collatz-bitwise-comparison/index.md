+++
date = '2025-03-15T19:26:19-06:00'
draft = false
title = 'Collatz Bitwise Comparison'
tags = ["math", "golang", "python"]
+++

**TL;DR** On Hacker News today I came across an interesting blog post called
[Collatzeral Damage: Bitwise and Proof Foolish](https://soatok.blog/2025/01/06/collatzeral-damage-bitwise-and-proof-foolish/).
It compared going through the sequence in a base-10 vs bitwise fashion. I was
curious about the performance differences, so I did the comparison in both
Go and Python. This post describes the results of that comparison.

# Methodology

Both of the programs for Go and Python were extremely simple, and I put them
in this GitHub repo: https://github.com/aldernero/collatz-bitwise-blog-post

The key parts of each are below:

**Go**
```go
func collatzRecursiveTraditional(n uint64) {
	if n <= 1 {
		return
	}
	if n%2 == 1 {
		collatzRecursiveTraditional(3*n + 1)
	} else {
		collatzRecursiveTraditional(n / 2)
	}
}

func collatzRecursiveBitwise(n uint64) {
	if n <= 1 {
		return
	}
	if (n & 1) == 1 {
		collatzRecursiveBitwise(((n << 1) | 1) + n)
	} else {
		collatzRecursiveBitwise(n >> 1)
	}
}
```

**Python**
```python
def collatz_recur_trad(n):
    if n == 1:
        return
    if n % 2 == 1:
        collatz_recur_trad(3*n + 1)
    else:
        collatz_recur_trad(n//2)

def collatz_recur_bit(n):
    if n == 1:
        return
    if n & 1:
        collatz_recur_bit(((n >> 1) | 1) + n)
    else:
        collatz_recur_bit(n >> 1)

```

The blog post covers a recursive approach, but in testing I also used an
iterative approach. I ran all 4 combinations (traditional && bitwise, recursive && iterative)
for both languages. I calculated the time to traverse the Collatz sequence for
integers from 1 to 10 million.

# Results

The table below shows the elapsed time for all combinations.

| Method      | Approach  | Go   | Python |
| ----------- | --------- | ---- | ------ |
| Traditional | recursive | 7.4s | 147.2s |
| Bitwise     | recursive | 7.2s | 29.8s  |
| Traditional | iterative | 2.4s | 101.8s |
| Bitwise     | iterative | 2.4s | 24.5s |

The results were mostly expected

- iterative is faster than recursive
- For Go the performance was similar for bitwise and traditional approaches, but for Python bitwise is faster
- Golang is faster than Python (duh)

I think the first point is expected since iterative is closer to what the CPU
actually does to perform the calculation, and uses less memory.

At first I wasn't
sure what the performance difference between traditional and bitwise would be for Go,
but given that it's a compiled language I suppose all the base-10 arithmetic
operations get turned into bitwise operations during compilation? :shrug: For Python it makes
sense that bitwise would be faster since it's an interpreted language and using bitwise
operations skips a step for the intepreter to do the conversion.

For Go we can isolate the comparison further by benchmarking the individual
parts of the calcualtion:

```go
func BenchmarkCollatzOddTraditional(b *testing.B) {
	for i := 0; i < b.N; i++ {
		_ = 3*i + 1
	}
}

func BenchmarkCollatzEvenTraditional(b *testing.B) {
	for i := 0; i < b.N; i++ {
		_ = i / 2
	}
}

func BenchmarkCollatzOddBitwise(b *testing.B) {
	for i := 0; i < b.N; i++ {
		_ = ((i << 1) | 1) + i
	}
}

func BenchmarkCollatzEvenBitwise(b *testing.B) {
	for i := 0; i < b.N; i++ {
		_ = i >> 1
	}
}
```

Running the benchmark shows:

```bash
goos: linux
goarch: amd64
pkg: collatzBitwise
cpu: AMD Ryzen 7 3700X 8-Core Processor             
BenchmarkCollatzOddTraditional
BenchmarkCollatzOddTraditional-16     	1000000000	         0.2551 ns/op
BenchmarkCollatzEvenTraditional
BenchmarkCollatzEvenTraditional-16    	1000000000	         0.2614 ns/op
BenchmarkCollatzOddBitwise
BenchmarkCollatzOddBitwise-16         	1000000000	         0.2528 ns/op
BenchmarkCollatzEvenBitwise
BenchmarkCollatzEvenBitwise-16        	1000000000	         0.2518 ns/op
```

For Python the results were mixed:

- *Expected*: iterative was faster that recursive
- *Expected*: bitwise was faster than traditional for the recursive approach
- *Surprising*: bitwise was *slower* than traditional for the iterative approach

I'm not an expert (translation: I know nothing) in how the Python interpreter
works, but I can't think of why 1) bitwise would be slower that traditional and 
2) recursive would be faster than iterative for the bitwise approach. Double :shrug:

# Summary

This was a fun little exercise. I would be interested to see comparisons for other languages, 
but I only had the patience to test these two. If you test some other language, let me know
on one of the social platforms.

# References

1. https://news.ycombinator.com/item?id=43375353 - Hacker News article discussing the blog post
2. [Collatzeral Damage: Bitwise and Proof Foolish](https://soatok.blog/2025/01/06/collatzeral-damage-bitwise-and-proof-foolish/) - blog post that inspired this post
3. https://github.com/aldernero/collatz-bitwise-blog-post - GitHub repo with the code used in this post

