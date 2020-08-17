+++
title = "All you need to know about Learning Tests"
description = "Understand libraries and protect your code"
tags = [
    "go",
    "golang",
    "tests",
    "development",
]
date = 2020-02-13T07:13:50Z
author = "Lorenzo Peppoloni"
+++

Picture this scenario:
You have to solve a new task, a new amazing coding problem, after some googling, you find a library that solve part of the problem for you. Great!
You write your code using the library, you write a test...and

FAIL!

You tweak the code a bit...FAIL!

I think we all went through this multiple times during our developer career.

What is happening is that, you found a new library to implement a certain behaviour that you want, you think you understood the library (but you have this bugging feeling that maybe you did not), you think you know how to use it for your particular use case (but you have this bugging feeling that maybe you do not).

What's a good approach? There is one simple answer: Learning tests

## What is a learning test?

A learning test is a test you write to test your understanding of a third party API library.
You basically write some tests in which you use the library as you will do in your production code and you check that the behaviour is what you expect.

The point here is that you are NOT testing the library (it should have its own tests), you are testing your understanding of it.

Why you should write learning tests?

An alternative would be to perform your own experiments using the library and then, when you are sure about its behavior, just use it in the production code.

While this may suffice, there are indeed several advantages in writing your "experiments" as actual tests.
* You would write the experiments anyway, so you are not adding any coding overhead.
* Learning tests protect your code against changes in the library itself. If a new version is released where a behaviour (or interface) is changed, you will immediately see your tests fail.  This will prevent you hours of painful debugging, only to understand that you're using a version of the library that is not compatible anymore with your code.

Let's make an example of a learning test

**Disclaimer**: *the example is trivial and probably everything can be solved beforehand reading the documentation accurately.*

Let's say we have a data structure `myStructWithTime` abstracting some data with a timestamp and we want to write a function to search by timestamp in an slice of our data structure.

After some research we encounter the [sort](https://golang.org/pkg/sort/) package in GO and we decide to give a try to its `Search` function. The package provides functionalities to sort slices and user-defined collections.

After a little bit of digging in the documentation, we think we got the mechanism. 
We write our search function

```go
// MyStructWithTime a structure with time.
type MyStructWithTime struct {
	foo       int
	timestamp time.Time
}

func findInStruct(in []MyStructWithTime, query time.Time) int {
	i := sort.Search(len(in), func(i int) bool {
		return in[i].timestamp.After(query)
	})
	if i < len(in) && in[i].timestamp.Equal(query) {
		return i
	}

	return -1
}
```


We then write a test in which we use the library in the same way we would in our production code.
First, it is not clear for us if the slice must be already sorted before using `sort.Search`, so we write a test and see what happens.

```go
earlier := time.Date(2020, time.January, 1, 2, 1, 0, 0, time.UTC)
later := time.Date(2020, time.January, 1, 5, 1, 0, 0, time.UTC)

testcases := []struct {
  name     string
  input    []MyStructWithTime
  query    time.Time
  expected int
}{
  {
    name: "not_sorted",
    input: []MyStructWithTime{
      {timestamp: later},
      {timestamp: earlier},
    },
    query:    earlier,
    expected: 1,
  },
}
````

You run the test and the result is:
```
--- FAIL: TestSort (0.00s)
    --- FAIL: TestSort/not_sorted (0.00s)
expected 1, got -1
FAIL
exit status 1
FAIL	0.005s
Error: Tests failed.
```
Probably we are doing something, wrong, probably the slice need to be already sorted, so we change the struct to

```go
{
    name: "sorted",
    input: []MyStructWithTime{
      {timestamp: earlier},
      {timestamp: later},
    },
    query:    earlier,
    expected: 0,
},

```

and we re-run the test
```
--- FAIL: TestSort (0.00s)
    --- FAIL: TestSort/not_sorted (0.00s)
expected 1, got -1
FAIL
exit status 1
FAIL	0.005s
Error: Tests failed.
```
again...

There must be something that we are missing here…We dig a bit more into the documentation, especially in the time package documentation, and we discover that After is not inclusive. From the sort documentation we got that we need to test for `>=` in a case of ascending sorted slice...Perfect!

Let's fix the function
```go
func findInStruct(in []MyStructWithTime, query time.Time) int {
	i := sort.Search(len(in), func(i int) bool {
		return in[i].timestamp.After(query) || in[i].timestamp.Equal(query)
	})
	if i < len(in) && in[i].timestamp.Equal(query) {
		return i
	}

	return -1
}
```

we hit the run button...and
```
Running tool: /usr/local/bin/go test -timeout 30s -run ^(TestSort)$

PASS
ok  	    0.005s
Success: Tests passed.
```

Success!!

We understood how we should use the library, and in the meantime we learnt a great deal about the `sort` and `time` packages.

At this point, the test can be factored into two test cases, which will be added to our test code base:

1) A test expecting failure for an array which is not sorted.
2) A working test where we put everything together.

These three tests will make sure that, if something changes in the `sort.Search`, we will be immediately notified by a test failure.

---

*Conclusions: Anytime you are facing a new library, do not limit yourself to write some experimental code to understand its use. A better approach is to write learning tests in which you use the library as you would do in your production code. In this way you'll test your actual understanding of the library and you'll protect your code from disruptive changes from third parties.*