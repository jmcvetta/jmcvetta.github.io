---
layout: post
title: "continuous integration for go code"
date: 2013-08-30 12:50
comments: true
categories:
published: false
---

Using a [continuous integration](http://en.wikipedia.org/wiki/continuous_integration)
pipeline has become an increasingly standard practice at modern software
companies.

Let's look at how to set up a ci pipeline for open source software written in
[go](http://golang.org).  All the services we will use are free (like beer)
when used for free (like freedom) software projects.


# Create a new project on Github

Start by creating a [new repository on github](https://github.com/new), naming
it `foofinder`.  Now clone the repo to your local machine:

``` text
$ git clone git@github.com:jmcvetta/foofinder.git # substitute your username
cloning into 'foofinder'...
host key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48
+--[ rsa 2048]----+
|        .        |
|       + .       |
|      . b .      |
|     o * +       |
|    x * s        |
|   + o o . .     |
|    .   e . o    |
|       . . o     |
|        . .      |
+-----------------+

remote: counting objects: 8, done.
remote: compressing objects: 100% (7/7), done.
remote: total 8 (delta 2), reused 4 (delta 1)
receiving objects: 100% (8/8), 12.87 kib, done.
resolving deltas: 100% (2/2), done.

$ cd foofinder/
```

Create a file `foofinder.go` with the following content:

## Write some code

``` go
package foofinder

import "errors"

func IsItFoo(word string) (bool, error) {
	switch word {
	case "foo":
		return true, nil
	case "bar":
		err := errors.New("Oh noooooo, it's bar!")
		return false, err
	}
	return false, nil
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
=== run TestIsItFoo
--- pass: TestIsItFoo (0.00 seconds)
pass
ok  	github.com/jmcvetta/foofinder	0.009s
```

## Test Coverage

We can use [`gocov`](http://github.com/axw/gocov) to ananlyze how much of our
codebase is covered by tests.

Install `gocov`:

``` text
$ go get -v github.com/axw/gocov/gocov
github.com/axw/gocov/gocov
```

`gocov` has several commands.   `gocov test` produces JSON output:


``` text
$ gocov test | python -mjson.tool  # use mjson.tool to pretty print JSON output
ok  	github.com/jmcvetta/foofinder	0.010s
{
    "packages": [
        {
            "functions": [
                {
                    "end": 462,
                    "entered": 0,
                    "file": "/home/jason/work/go/src/github.com/jmcvetta/foofinder/foofinder.go",
                    "left": 0,
                    "name": "isitfoo",
                    "start": 272,
                    "statements": [
                        {
                            "end": 441,
                            "reached": 1,
                            "start": 315
                        },
                        {
                            "end": 360,
                            "reached": 1,
                            "start": 344
                        },
                        {
                            "end": 418,
                            "reached": 0,
                            "start": 376
                        },
                        {
                            "end": 438,
                            "reached": 0,
                            "start": 421
                        },
                        {
                            "end": 460,
                            "reached": 0,
                            "start": 443
                        }
                    ]
                }
            ],
            "name": "github.com/jmcvetta/foofinder"
        }
    ]
}

```

The other `gocov` commands are used to interpret the json output.  `gocov
report` produces a human-readable summary:

``` bash
$ gocov test | gocov report
ok  	github.com/jmcvetta/foofinder	0.011s

github.com/jmcvetta/foofinder/foofinder.go	 IsItFoo 40.00% (2/5)
github.com/jmcvetta/foofinder			 ------- 40.00% (2/5)
```

For a more detailed look at your code's coverage, use `gocov annotate`.  Note
the trailing `-`:

``` bash
$ gocov test | gocov annotate -
ok  	github.com/jmcvetta/foofinder	0.011s

 9     	func IsItFoo(word string) (bool, error) {
10     		switch word {
11     		case "foo":
12     			return true, nil
13     		case "bar":
14 miss			err := errors.New("oh noooooo, it's bar!")
15 miss			return false, err
16     		}
17 miss		return false, nil
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


# Travis CI

Travis CI is the most popular hosted continuous integration service for open
source projects.  It is tightly integrated with Github.  Travis results are
automatically displayed for pull requests, helping to ensure you never accept a
PR that fails its tests.

Travis itself is open source, and free for use by open source projects.
However it has some limitations for testing go code - in particular,
`gocov` cannot currently be used with Travis.

## Travis Configuration

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

The `before_script` section is required because our tests depend on `assert`,
but the `go get` command will not automatically fetch packages imported by
`*_test.go` files.

Add the travis configuration to git:

``` text
$ git add .travis.yml

$ git commit -m "travis configuration"
[master 3329dc4] travis configuration
 1 file changed, 8 insertions(+)
 create mode 100644 .travis.yml
```

## Activate Travis

Open https://travis-ci.org in your browser.  Click on "Sign in with Github", in
the upper right corner of the screen.  If you have never logged in to travis
before you will be prompted by Github to grant OAuth permissions to Travis.

{% img images/travis-signin.png travis sign-in %}

Once you are logged in, click on your username in the upper right corner, and
choose "Accounts" from the dropdown.

{% img images/travis-accounts.png travis accounts %}

Travis will display a list of all your Github repositories.  locate `foofinder`,
and click the OFF/ON toggle beside it.  The toggle will slide to "ON".

{% img images/travis-repos.png travis repositories %}

Now we push our commits to Github, which will automatically kick off a
Travis build:

``` text
$ git push
host key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48
+--[ rsa 2048]----+
|        .        |
|       + .       |
|      . b .      |
|     o * +       |
|    x * s        |
|   + o o . .     |
|    .   e . o    |
|       . . o     |
|        . .      |
+-----------------+

counting objects: 4, done.
delta compression using up to 4 threads.
compressing objects: 100% (3/3), done.
writing objects: 100% (3/3), 413 bytes, done.
total 3 (delta 1), reused 0 (delta 0)
to git@github.com:jmcvetta/foofinder.git
   80f9eb6..3329dc4  master -> master
```

In your browser, click on the Travis logo in the upper left corner of the
screen to return to the travis home page.  You should now see `foofinder` in
the "my repositories" sidebar at the left of the screen.  Click on it to see
the results of your test run.

{% img images/travis-build.png travis build results %}


# Drone.io

[Drone](http://drone.io) is a hosted continuous integration service, in some
ways similar to Travis CI.  Drone provides a more "vanilla"  build
environment than Travis, that works great with `gocov` and friends.  Although
Drone itself is proprietary, it is free for use by open source projects.  The
backend of drone is written in go.

Unlike Travis, Drone does not require a configuration file in the repository.
Instead it is configured through its web interface.

Start by opening http://drone.io in your browser.  Click "Login" in the upper
right of the screen.

{% img images/drone-login.png Login to Drone %}

On the login page, choose Github from the list of third-party login providers.
If you have never logged in to Drone before, Github will prompt you to grant
OAuth permissions.

{% img images/drone-login-provider.png Drone Login Providers %}

Once you are logged in, click on the "New Project" link at the top of the page.

{% img images/drone-new-project.png New Project %}

On the repository setup page, select Github.

{% img images/drone-repo-setup.png Repository Setup %}

Locate `foofinder` in the list of your repositories, and click "Select".

{% img images/drone-select-repo.png Select Repository %}

Choose Go from the project setup menu.

{% img images/drone-golang.png Setup Go project %}

You will be taken to the build script screen.

It is necessary to `go get` the `assert` package required by our tests.  Replace
the default build script with this:

``` text
go get
go build
go get github.com/bmizerany/assert
go test -v
```

{% img images/drone-setup-initial.png Initial Drone build script %}

Click "Save" and you will be taken to the Settings screen.  No changes need
to be made here.

{% img images/drone-settings-initial.png Initial Drone settings page %}

Click "Build Now" to kick off the first build!  Every time a new commit is
pushed to Github, Drone will automatically start a build.  If everything is
setup correctly your build will complete successfully.

{% img images/drone-build-success.png First Drone build was successful %}



# Coveralls.io

[Coveralls](http://coveralls.io) is a hosted code coverage analysis tool.  It is
free for use by open source projects.


## Configure Coveralls

Start by opening http://coveralls.io in your browser.  Click "SIGN IN WITH
GITHUB" in the upper right corner.  If you have never logged in to Coveralls
before, Github will prompt you to grant OAuth permissions to Coveralls.

{% img images/coveralls-home.png Coveralls Sign In %}

Once you are logged in, click on "Add Repo".

{% img images/coveralls-signed-in.png Click on Add Repo %}

Coveralls does not automatically refresh its list of your Github repositories.
You will need to manually sync it whenever you add a new repo on Github.  Click
"SYNC GITHUB REPOS".  The operation may take several seconds.

{% img images/coveralls-sync.png Sync Repos %}

Locate `foofinder` in the list of repositories, and click the OFF/ON toggle.  It
will slide over to ON, and a "VIEW ON COVERALLS" button will appear.  Sometimes
it may be necessary to refresh the page to see the button.

{% img images/coveralls-toggle.png Toggle Coveralls for the project %}

Click "VIEW ON COVERALLS" and you will be taken to the setup page.  Locate the
REPO TOKEN in the TECHNICAL DETAILS section.  Copy the token to your clipboard.

{% img images/coveralls-tech-details.png Locate the Repo Token %}


## Add `goveralls` to Drone config

Now we need to push some test coverage data to Coveralls.  We will do this by
adding `goveralls`, a Coveralls client for Go code, to our Drone build script.
We must use Drone because `goveralls` calls `gocov`, which is not supported on
Travis CI.

Return to the Drone settings for `foofinder`, and update the Commands section
to call `goveralls`:

``` text
go get
go build
go get github.com/bmizerany/assert
go test -v

go get -v github.com/axw/gocov/gocov
go get -v github.com/mattn/goveralls
goveralls -v -service drone.io $COVERALLS_TOKEN
```

Then paste your Coveralls Repo Token into the Environment Variables section
like this:

``` text
COVERALLS_TOKEN=paste_your_token_here
```

Note, the Environment Variables of a Drone project are not visible to the
public, but the Commands are.

{% img images/drone-coveralls-settings.png Configure Drone to work with Coveralls %}

Click "Save", then click "Build Now" to kick off a new build.  If everything
is set up correctly, the build will display a Coveralls job link when it
completes.

{% img images/drone-coveralls-success.png Successful push to Coveralls %}

Refresh your project's Coveralls page, and you will see code coverage data!

{% img images/coveralls-first-build.png First build %}

Click on the build number - in this case, "#1" - to explore coverage in the
build.

{% img images/coveralls-build-details.png Build Details %}

By clicking on an individual file name, you can view color coded code coverage
for that file.

{% img images/coveralls-file-details.png Source File Details %}
