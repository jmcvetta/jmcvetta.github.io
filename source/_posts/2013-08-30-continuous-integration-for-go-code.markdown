---
layout: post
title: "Continuous Integration for Go code"
date: 2013-08-30 12:50
comments: true
categories:
published: false
---

Using a [continuous integration](http://en.wikipedia.org/wiki/Continuous_integration)
pipeline has become an increasingly standard practice at modern software
companies.

Let's look at how to set up a CI pipeline for open source software written in
[Go](http://golang.org).  All the services we will use are free (like beer) for
for use by Free (like Freedom) Software projects.


# Create a new project on Github

Start by creating a [new repository on Github](https://github.com/new), naming
it `foofinder`.

Create a file `foofinder.go` with the following content:

## Write some code

``` go
package foofinder

import (
	"github.com/bmizerany/assert"
	"testing"
)

func TestIsItFoo(t *testing.T) {
	word := "foo"
	foo, err := IsItFoo(word)
	if err != nil {
		t.Fatal(err)
	}
	assert.Equal(t, true, foo)
}
```


## Write a test

Create a file `foofinder_test.go` with this content:

``` go
package foofinder

import (
	"github.com/bmizerany/assert"
	"testing"
)

func TestIsItFoo(t *testing.T) {
	word := "foo"
	foo, err := IsItFoo(word)
	if err != nil {
		t.Fatal(err)
	}
	assert.Equal(t, true, foo)
}
```

Now let's run our test and see if it worked:

``` bash
$ go test -v
=== RUN TestIsItFoo
--- PASS: TestIsItFoo (0.00 seconds)
PASS
ok  	github.com/jmcvetta/foofinder	0.009s
```


## Test coverage

We can use [`gocov`](http://github.com/axw/gocov) to ananlyse how much of our
codebase is covered by tests.

Install `gocov`:

``` bash
$ go get -v github.com/axw/gocov/gocov
github.com/axw/gocov/gocov
```

`gocov` has several commands.   `gocov test` produces JSON output:

``` bash
$ gocov test | python -mjson.tool  # Use mjson.tool to pretty print JSON output
ok  	github.com/jmcvetta/foofinder	0.010s
{
    "Packages": [
        {
            "Functions": [
                {
                    "End": 462,
                    "Entered": 0,
                    "File": "/home/jason/work/go/src/github.com/jmcvetta/foofinder/foofinder.go",
                    "Left": 0,
                    "Name": "IsItFoo",
                    "Start": 272,
                    "Statements": [
                        {
                            "End": 441,
                            "Reached": 1,
                            "Start": 315
                        },
                        {
                            "End": 360,
                            "Reached": 1,
                            "Start": 344
                        },
                        {
                            "End": 418,
                            "Reached": 0,
                            "Start": 376
                        },
                        {
                            "End": 438,
                            "Reached": 0,
                            "Start": 421
                        },
                        {
                            "End": 460,
                            "Reached": 0,
                            "Start": 443
                        }
                    ]
                }
            ],
            "Name": "github.com/jmcvetta/foofinder"
        }
    ]
}

```

The other `gocov` commands are used to interpret the JSON output.  `gocov
report` produces a human-readable summary:

``` bash
$ gocov test | gocov report
ok  	github.com/jmcvetta/foofinder	0.011s

github.com/jmcvetta/foofinder/foofinder.go	 IsItFoo 40.00% (2/5)
github.com/jmcvetta/foofinder			 ------- 40.00% (2/5)
```

For a more detailed look at your code's coverage, use `gocov annotate`:

``` bash
$ gocov test | gocov annotate -
ok  	github.com/jmcvetta/foofinder	0.011s

 9     	func IsItFoo(word string) (bool, error) {
10     		switch word {
11     		case "foo":
12     			return true, nil
13     		case "bar":
14 MISS			err := errors.New("Oh noooooo, it's bar!")
15 MISS			return false, err
16     		}
17 MISS		return false, nil
18     	}
```


# Set up Travis CI

## Why Travis rocks

## Why Travis sucks


# Set up Drone.io


# Set up Coveralls

## Add `goveralls` to Drone config.
