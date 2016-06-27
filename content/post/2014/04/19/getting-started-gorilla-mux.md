---
date: 2014-04-19T16:05:12Z
title: "Getting Started with Gorilla MUX"
description: "Quick intro to the Gorilla MUX package."
tags: [ "golang", "http" ]
Section: blog
categories:
  - "Development"
slug: "getting-started-gorilla-mux"
---

This is going to be short and sweet.  I have been looking at getting into Go for awhile but lacked incentive.  That has, as of a week ago, changed and it's full speed ahead.

For this I will be using [Gorilla MUX](http://www.gorillatoolkit.org/pkg/mux).  If you haven't taken a look at the Gorilla toolkit yet I highly recommend it.  I really enjoy using it, the use as individual components really resonates with me and my style of coding.

### Setup
Setup for this is pretty fast, `go get github.com/gorilla/mux`

### The Meat and Potatos
The `main.go` for this is pretty simple, but if you haven't used [net/http](http://golang.org/pkg/net/http/) from the Golang standard library you might be scratching your head trying to figure out what's going on.

##### Example main.go
    package main

    import (
        "fmt"
        "github.com/gorilla/mux"
        "net/http"
    )

    func ArticleHandler(w http.ResponseWriter, r *http.Request) {
        fmt.Println("Article URI")
    }

    func main() {
        // Create a new Router instance
        router := mux.NewRouter()

        // Add the URI /article to be handled by the ArticleHandler method
        router.HandleFunc("/article", ArticleHandler)
        
        // Add the URI / to be handled by a closure
        router.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
            fmt.Println("Root URI")
        })

        // Pass our router to net/http
        http.Handle("/", router)
        
        // Listen and serve on localhost:8080
        http.ListenAndServe(":8080", nil)
    }

Alright, that's cool so what do we do with it?

Well, sounds like a great way build a back end service and a separate front end application using something like AngularJS.

Next article I will go into more detail.  Until then, you can check out the code for this series here: [HelloBlog](https://github.com/taion809/go-helloblog)

As always, if there are errors or you have a suggestion send me a message.
