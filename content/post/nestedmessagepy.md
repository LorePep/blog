+++
title = "Proto nested messages and repeated fields in Python"
description = "How to create repeated nested objects using Python"
tags = [
    "proto",
    "python",
    "development",
]
date = 2020-02-04T20:13:50Z
author = "Lorenzo Peppoloni"
+++

Today I was having some problems populating a proto repeated message in Python with a nested message definition, and it took me a while to figure out how to do it.

In reality it is pretty simple. Let's make an example.

```
syntax = "proto3";

package test;

message Trajectory2d {
    message Point2d {
      float x = 1;
      float y = 2;
    }
    repeated Point2d points = 1;
  }
```

Let's save our `test.proto` and generate the Python code.
```bash
protoc --proto_path=. --python_out=. test.proto
```

Now if we want to create an element of `Trajectory2d` type add points to it, we can just use the `add()` function, which creates a new message object, appends it to the list of repeated objects, and returns it for the caller to fill, plus it forwards keyword arguments to the class.

```python
from test_pb2 import Trajectory2d

trajectory = Trajectory2d()
trajectory.points.add(x=10, y=30)
trajectory.points.add(x=14, y=22)

assert len(trajectory.points) == 2

assert trajectory.points[0].x == 10
assert trajectory.points[0].y == 30

assert trajectory.points[1].x == 14
assert trajectory.points[1].y == 22
```
