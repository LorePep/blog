+++
title = "Table-driven tests in Python"
description = "Some ideas about table-driven tests from Go to Python."
tags = [
    "go",
    "golang",
    "python",
    "tests",
    "development",
]
date = 2020-02-02T02:13:50Z
author = "Lorenzo Peppoloni"
+++


[Table-driven tests](https://dave.cheney.net/2013/06/09/writing-table-driven-tests-in-go) are an elegant and functional way to unittest your functions in GO. Let's see some ideas on how to introduct this same testing pattern in Python.


## What are table-driven tests

One thing I really love about GO is table-driven tests. If you are not familiar with them, table-driven tests are a very elegant way to write unittests for your code. The basic idea is that you write a list of named test cases, defining the input and the expected output for each test case, then you loop over the cases, run your function and check that the actual output is equal to the expected one.

An example in Go looks like this, let's imagine we want to test a sorting function we wrote:

```
func TestMySort(t *testing.T) {
	testcases := []struct {
		name     string
		input    []float64
		expected []float64
	}{
		{
			name:     "empty_slice",
			input:    []float64{},
			expected: []float64{},
		},
		{
			name:     "already_sorted",
			input:    []float64{1, 4, 6, 8},
			expected: []float64{1, 4, 6, 8},
		},
		{
			name:     "not_sorted",
			input:    []float64{1, 8, 3, 5},
			expected: []float64{1, 3, 5, 8},
		},
	}

	for _, tt := range testcases {
		t.Run(tt.name, func(t *testing.T) {
			actual := mySort(tt.input)
			assertEqualSlices(t, tt.expected, actual)
		})
	}
}
```

As you can see, we wrote three named test cases (empty slice in input, input already sorted and input not sorted). The final part of the code is just looping and asserting that for each test case we got the expected value.

What I think it's really great about table-driven tests is that they allow you to naturally write very modular and concise tests, focusing on test data and expected behaviours. I also find that from a psychological viewpoint, they help you reasoning more in depth about test cases and in general be more thoughtful on what input could break your code.

When I switch to Python, I always feel like I'm missing table-driven tests and I always end up finding Pythonic ways of implementing them. 

Here a couple ideas I came up with.

## Python dicts

One simple and yet effective way of implementing table-driven tests in Python is using dicts. Let's see an example, with the same sorting function.

```python
import unittest

class TestMySort(unittest.TestCase):
    def test_my_sort(self):
        testcases = [
            {"name": "empty_slice", "input": [], "expected": [],},
            {
                "name": "already_sorted",
                "input": [1, 4, 6, 8],
                "expected": [1, 4, 6, 8],
            },
            {"name": "not_sorted", "input": [1, 8, 3, 5], "expected": [1, 3, 5, 8],},
        ]

        for case in testcases:
            actual = my_sort(case["input"])
            self.assertListEqual(
                case["expected"],
                actual,
                "failed test {} expected {}, actual {}".format(
                    case["name"], case["expected"], actual
                ),
            )

```

The main advantage of this approach is that it's simple, understandable and it is compatible with every Python version.

The main problem I see is that there is not much protection around the `testcase` datastructure. You could make a mistake and the dictionaries could have different unexpected keys or different types. Typing could be enforced, but still the best you can do is defining the testcases `List[Dict[str, Any]]` which is not very strict.

## Data Class
If you are using Python `3.7` you can use [data classes](https://docs.python.org/3/library/dataclasses.html). A data class is a class containing mainly data, the advantage is that it comes with already pre-defined methods, such as __init__() and __repr__() making you save time when coding. 

Let's see how can we use them for table-drivent tests.

```python
import unittest
from dataclasses import dataclass
from typing import List


class TestMySort(unittest.TestCase):
    def test_my_sort(self):
        @dataclass
        class TestCase:
            name: str
            input: List[float]
            expected: List[float]

        testcases = [
            TestCase(name="empty_slice", input=[], expected=[]),
            TestCase(name="already_sorted", input=[1, 4, 6, 8], expected=[1, 4, 6, 8]),
            TestCase(name="not_sorted", input=[1, 8, 3, 5], expected=[1, 3, 5, 8]),
        ]

        for case in testcases:
            actual = my_sort(case.input)
            self.assertListEqual(
                case.expected,
                actual,
                "failed test {} expected {}, actual {}".format(
                    case.name, case.expected, actual
                ),
            )
```

Overall using data classes gives you a cleaner solution compared to dicts, since you can easily enforce typing.

* * *

In this article we quickly had a look at what are table-driven tests in GO and why they are a nice feature. We then explored possible solutions to impolement table-driven tests in Python.
