---
layout: post
title: "Continuous Integration for Go code"
date: 2013-08-30 12:50
comments: true
categories: [Go, Continuous Integration]
---

{% blockquote Martin Fowler http://www.martinfowler.com/articles/continuousIntegration.html Continuous Integration %}
Continuous Integration is a software development practice where members of a
team integrate their work frequently, usually each person integrates at least
daily - leading to multiple integrations per day. Each integration is verified
by an automated build (including test) to detect integration errors as quickly
as possible. Many teams find that this approach leads to significantly reduced
integration problems and allows a team to develop cohesive software more
rapidly.
{% endblockquote %}

Using a [continuous integration](http://en.wikipedia.org/wiki/continuous_integration)
(CI) pipeline is a good practice for any software project.  CI lets a developer
know immediately if they commit changes that break any unit tests, or decrease
test coverage.

Let's look at how to set up a CI pipeline for open source software written in
[Go](http://golang.org).  All the services mentioned here are free (like beer)
when used for Free (like freedom) Software projects.


# Create project

Start by creating a [new repository on Github](https://github.com/new), naming
it `foofinder`.  Now clone the repo to your local machine:

``` bash
git clone git@github.com:jmcvetta/foofinder.git # substitute your username
cd foofinder/ # Remaining commands assume they are in the "foofinder" folder

```


## Write some code

Create a file `foofinder.go` with the following content:

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
git add foofinder.go foofinder_test.go
git commit -m "trivial program"
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
git add .travis.yml
git commit -m "travis configuration"
```

## Activate Travis

Open https://travis-ci.org in your browser.  Click on "Sign in with Github", in
the upper right corner of the screen.  If you have never logged in to travis
before you will be prompted by Github to grant OAuth permissions to Travis.

{% img /images/go_ci/travis-signin.png travis sign-in %}

Once you are logged in, click on your username in the upper right corner, and
choose "Accounts" from the dropdown.

{% img /images/go_ci/travis-accounts.png travis accounts %}

Travis will display a list of all your Github repositories.  locate `foofinder`,
and click the OFF/ON toggle beside it.  The toggle will slide to "ON".

{% img /images/go_ci/travis-repos.png travis repositories %}

Now we push our commits to Github, which will automatically kick off a
Travis build:

``` text
$ git push
Host key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48
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

{% img /images/go_ci/travis-build.png travis build results %}


## Status badge

After all this work comes the reward:  a nice, shiny green status badge!

Click on the gear icon and choose "Status Images":

{% img /images/go_ci/travis-status-images-link.png %}

Copy the Markdown link from the ensuing dialog.

{% img /images/go_ci/travis-status-images.png %}

Paste the Markdown into the `README.md` file:

``` text
foofinder
=========

Silly example for a blog post

[![Build Status](https://travis-ci.org/jmcvetta/foofinder.png?branch=master)](https://travis-ci.org/jmcvetta/foofinder)
```


Commit and push the changes:

```
git add README.md
git commit -m "Travis status badge"
git push
```

Now when you look at the project's Github page, you will see a status badge
that is automatically updated whenever a new build is run on Travis.

{% img /images/go_ci/travis-status-badge.png Travis badge on Github %}



# Drone.io

[Drone](http://drone.io) is a hosted continuous integration service, in some
ways similar to Travis CI.  Drone provides a more "vanilla"  build
environment than Travis, that works great with `gocov` and friends.  Although
Drone itself is proprietary, it is free for use by open source projects.  The
backend of Drone is written in Go.

Unlike Travis, Drone does not require a configuration file in the repository.
Instead it is configured through its web interface.

Start by opening http://drone.io in your browser.  Click "Login" in the upper
right of the screen.

{% img /images/go_ci/drone-login.png Login to Drone %}

On the login page, choose Github from the list of third-party login providers.
If you have never logged in to Drone before, Github will prompt you to grant
OAuth permissions.

{% img /images/go_ci/drone-login-provider.png Drone Login Providers %}

Once you are logged in, click on the "New Project" link at the top of the page.

{% img /images/go_ci/drone-new-project.png New Project %}

On the repository setup page, select Github.

{% img /images/go_ci/drone-repo-setup.png Repository Setup %}

Locate `foofinder` in the list of your repositories, and click "Select".

{% img /images/go_ci/drone-select-repo.png Select Repository %}

Choose Go from the project setup menu.

{% img /images/go_ci/drone-golang.png Setup Go project %}

You will be taken to the build script screen.

It is necessary to `go get` the `assert` package required by our tests.  Replace
the default build script with this:

``` text
go get
go build
go get github.com/bmizerany/assert
go test -v
```

{% img /images/go_ci/drone-setup-initial.png Initial Drone build script %}

Click "Save" and you will be taken to the Settings screen.  No changes need
to be made here.

{% img /images/go_ci/drone-settings-initial.png Initial Drone settings page %}

Click "Build Now" to kick off the first build!  Every time a new commit is
pushed to Github, Drone will automatically start a build.  If everything is
setup correctly your build will complete successfully.

{% img /images/go_ci/drone-build-success.png First Drone build was successful %}


## Status badge

In your Drone project settings, click on "Status Badges".

{% img /images/go_ci/drone-status-markdown.png Markdown for status badge %}

Copy the Markdown, and paste it into your `README.md` file:

``` text
foofinder
=========

Silly example for a blog post

[![Build Status](https://travis-ci.org/jmcvetta/foofinder.png?branch=master)](https://travis-ci.org/jmcvetta/foofinder)
[![Build Status](https://drone.io/github.com/jmcvetta/foofinder/status.png)](https://drone.io/github.com/jmcvetta/foofinder/latest)
```

Commit and push the changes:

``` text
git add README.md
git commit -m "Drone status badge"
git push
```

The Drone status badge is now visible on Github!

{% img /images/go_ci/drone-status-badge.png Drone status badge %}


# Coveralls.io

[Coveralls](http://coveralls.io) is a hosted code coverage analysis tool.  It is
free for use by open source projects.


## Configure Coveralls

Start by opening http://coveralls.io in your browser.  Click "SIGN IN WITH
GITHUB" in the upper right corner.  If you have never logged in to Coveralls
before, Github will prompt you to grant OAuth permissions to Coveralls.

{% img /images/go_ci/coveralls-home.png Coveralls Sign In %}

Once you are logged in, click on "Add Repo".

{% img /images/go_ci/coveralls-signed-in.png Click on Add Repo %}

Coveralls does not automatically refresh its list of your Github repositories.
You will need to manually sync it whenever you add a new repo on Github.  Click
"SYNC GITHUB REPOS".  The operation may take several seconds.

{% img /images/go_ci/coveralls-sync.png Sync Repos %}

Locate `foofinder` in the list of repositories, and click the OFF/ON toggle.  It
will slide over to ON, and a "VIEW ON COVERALLS" button will appear.  Sometimes
it may be necessary to refresh the page to see the button.

{% img /images/go_ci/coveralls-toggle.png Toggle Coveralls for the project %}

Click "VIEW ON COVERALLS" and you will be taken to the setup page.  Locate the
REPO TOKEN in the TECHNICAL DETAILS section.  Copy the token to your clipboard.

{% img /images/go_ci/coveralls-tech-details.png Locate the Repo Token %}


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

{% img /images/go_ci/drone-coveralls-settings.png Configure Drone to work with Coveralls %}

Click "Save", then click "Build Now" to kick off a new build.  If everything
is set up correctly, the build will display a Coveralls job link when it
completes.

{% img /images/go_ci/drone-coveralls-success.png Successful push to Coveralls %}

Refresh your project's Coveralls page, and you will see code coverage data!

Note the coverage percentage calculated by Coveralls sometimes disagrees with
that calculated by `gocov report`.  Afaik this has to do with how they count
"meaningful" lines of code.

{% img /images/go_ci/coveralls-first-build.png First build %}

Click on the build number - in this case, "#1" - to explore coverage in the
build.

{% img /images/go_ci/coveralls-build-details.png Build Details %}

By clicking on an individual file name, you can view color coded code coverage
for that file.

{% img /images/go_ci/coveralls-file-details.png Source File Details %}


## Status Badge

On your project's Coveralls page, click the "Get Badge URLs" link, just below
the color-coded "coverage" badge.  Copy the Markdown from the ensuing popup:

{% img /images/go_ci/coveralls-badge-markdown.png Badge markdown %}

Paste the Markdown into your `README.md` file:

``` text
foofinder
=========

Silly example for a blog post

[![Build Status](https://travis-ci.org/jmcvetta/foofinder.png?branch=master)](https://travis-ci.org/jmcvetta/foofinder)
[![Build Status](https://drone.io/github.com/jmcvetta/foofinder/status.png)](https://drone.io/github.com/jmcvetta/foofinder/latest)
[![Coverage Status](https://coveralls.io/repos/jmcvetta/foofinder/badge.png?branch=master)](https://coveralls.io/r/jmcvetta/foofinder?branch=master)
```

Commit and push the changes:

``` text
git add README.md
git commit -m "Coveralls status badge"
git push
```

The coverage badge is now visible on Github!

{% img /images/go_ci/coveralls-badge.png Coveralls status badge %}

Uh-oh, that badge is red.  Not-so-shiny.  :(  We better get to work improving
our test coverage!

