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
[Go](http://golang.org).  All the services we will use are free (like beer)
when used for Free (like freedom) Software projects.


# Create a new project on Github

Start by creating a [new repository on Github](https://github.com/new), naming
it `foofinder`.  Now clone the repo to your local machine:

``` text
$ git clone git@github.com:jmcvetta/foofinder.git # substitute your username
Cloning into 'foofinder'...
Host key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48
+--[ RSA 2048]----+
|        .        |
|       + .       |
|      . B .      |
|     o * +       |
|    X * S        |
|   + O o . .     |
|    .   E . o    |
|       . . o     |
|        . .      |
+-----------------+

remote: Counting objects: 8, done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 8 (delta 2), reused 4 (delta 1)
Receiving objects: 100% (8/8), 12.87 KiB, done.
Resolving deltas: 100% (2/2), done.

$ cd foofinder/
```

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

``` text
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

``` text
$ go get -v github.com/axw/gocov/gocov
github.com/axw/gocov/gocov
```

`gocov` has several commands.   `gocov test` produces JSON output:


``` text
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


## Commit to Github

Now we will commit our new files to the git repository:

``` text
$ git add foofinder.go foofinder_test.go

$ git commit -m "trivial program"
[master a09f2ab] trivial program
 2 files changed, 37 insertions(+)
 create mode 100644 foofinder.go
 create mode 100644 foofinder_test.go
```


# Set up Travis CI

Travis CI is the most popular hosted continuous integration service for open
source projects.  It is tightly integrated with Github.  However it has some
limitations for testing Go code - in particular, `gocov` cannot currently be
used with Travis.

Travis reads its configuration from a file in the repository root.  Create a
`.travis.yml` file with this content:

``` yaml
language: go
notificaitons:
  email:
    recipients: your.email@host.com
    on_success: change
    on_failure: always
before_script:
- go get github.com/bmizerany/assert
```

Add the Travis configuration to Git:

``` text
$ git add .travis.yml

$ git commit -m "Travis configuration"
[master 3329dc4] Travis configuration
 1 file changed, 8 insertions(+)
 create mode 100644 .travis.yml
```

## Activate Travis

Open https://travis-ci.org in your browser.  Click on "Sign in with Github", in
the upper right corner of the screen.  If you have never logged in to Travis
before you will be prompted by Github to grant OAuth permissions to Travis.

Once you are logged in, click on your username in the upper right corner, and
choose "Accounts" from the dropdown.  Travis will display a list of all your
Github repositories.  Locate `foofinder`, and click the OFF/ON toggle beside
it.  The toggle will slide to "ON".

Now we push our commits to Github, which will automatically kick off a
Travis build:

``` text
$ git push
Host key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48
+--[ RSA 2048]----+
|        .        |
|       + .       |
|      . B .      |
|     o * +       |
|    X * S        |
|   + O o . .     |
|    .   E . o    |
|       . . o     |
|        . .      |
+-----------------+

Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 413 bytes, done.
Total 3 (delta 1), reused 0 (delta 0)
To git@github.com:jmcvetta/foofinder.git
   80f9eb6..3329dc4  master -> master
```

In your browser, click on the Travis logo in the upper left corner of the
screen to return to the Travis home page.  You should now see `foofinder` in
the "My Repositories" sidebar at the left of the screen.  Clicking on it


# Set up Drone.io


# Set up Coveralls

## Add `goveralls` to Drone config.
