---
title: Exploration in golang
---

At work I've been using a lot of Scala. Since I've made the switch, I've seen a lot of posts on teams moving away from Scala:
* [CrowdStrike](http://jimplush.com/talk/2015/12/19/moving-a-team-from-scala-to-golang/)
* [Paul Dix](http://devslovebacon.com/conferences/bacon-2013/talks/why-node-and-scala-will-dry-up-go-will-drink-their-milkshake)
* [Yammer moving away from Scala](https://codahale.com/downloads/email-to-donald.txt)

The TLDR version of all 3 was that Scala is too expressive and unopinionated, leading to different coders having vastly different styles and eventually becoming cumbersome for a large team to handle. In effect, there's extra overhead for each programmer added on to the team.

I believe that the tools you use every day shape the fundamental way you approach and solve problems. So if the failure mode of Scala is being too *unopinionated* maybe learning an *opinionated* language like Golang will help me avoid some Scala pitfalls. Will this actually work? IDK.

## Installation on OSX
Install golang: https://golang.org/dl/

The OSX package installs to `/usr/local/go`, so add `/usr/local/go/bin` to the `PATH` in `~/.bashrc` or `~/.zhrc` file.

Golang requires that all the libraries & code be structured in a certain way:

```bash
~/jia/go
  /bin
    project1Binary
  /pkg
  /src
    /project1
    /project2
```

Where `~/jia/go` is my `$GOPATH`. `$GOPATH` has to be added into the path, otherwise there's a build error. The golang page [explains it in detail](https://golang.org/doc/code.html).

## First App

I warmed up with a few problems from [Project Euler](https://projecteuler.net/) to learn golang syntax. The last few times I've tried to learn a language from scratch I jump directly into a project, but then I spend too long context switching between both the project task and writing idiomatic JS/Python/Scala, etc.  

Then I made this [7minworkout app](https://github.com/jiahuang/7minworkout) that runs in the terminal. It has gifs like this:

![demo](https://raw.githubusercontent.com/jiahuang/7minworkout/master/workout.gif)

Ok, so maybe the app was mostly an exploration in emoticons. But in the process I ended up writing a really simple multi-line terminal animation library.

The majority of the logic is in 80 very spaced out lines.

Object creation looks like C. This declares an Animation struct that has an array of arrays with strings for the animation buffer:
```golang
type Animation struct {
  frames [][]string
  loop int
  delay time.Duration
  timeout chan bool
}
```

Animations can be instantiated with
```golang
s := &Animation{
    frames: {{"this is the first line"}, {"line 2 first frame", "line 2nd frame"}},
    loop: 2,
    delay: 500*time.Millisecond,
    timeout: make(chan bool),
  }

s.Start()  
```

The full source of the app is [here on github](https://github.com/jiahuang/7minworkout).
