---
title: 'A Make for URIs'
date: Mon, 16 Nov 2020 18:07:45 -0800
tags: ['Mk', 'Make']
---

`Make` has been a one of the key tools in my arsenal for gettings
things done. Although it was developed for compiling code, its
functionality can be generalized to any process that requires files
to be generated based on dependancies. 
I recommend you look at these [slides by Vince Buffalo](https://github.com/vsbuffalo/makefiles-in-bioinfo/blob/master/makefile-slides.pdf)
as a good introduction to using `make` for scientific workflows.
`Make` works by creating a 
dependancy graph of files and their prerequisites
using the last time the file was
modified as a way to determine if a file needs to be remade.
This general concept is great for 
reproducable scientific research or many other repeating
tasks and workflows. However it's not without it's flaws. 

Make's syntax is very obtuse using many shorthand variables like
to describe rules that make it difficult to start using. But 
even after leaning its syntax I've continually found one fatal flaw:
all of the files need to be local, on your current computer. 
This makes sense given its original function
of compiling code. However in data analysis and scientific workflows
we often have to interact with remote files on AWS S3 or files that
we download from a web resourse. These files don't have a time stamp
that `make` can use and so their presense completely breaks the
dependancy graph.

There are work arounds of course. I've used dummy empty files as a
way of keeping track of the last time a URL was downloaded or 
just downloaded all of the external files at the beginning and
end of an analysis, which is used in the popular 
[cookiecutter data science template](https://drivendata.github.io/cookiecutter-data-science/).
But both of these solutions are brittle and don't really solve the
problem.

I looked for a URL enabled make, one that could integrate in remote
files to the dependancy graph, but didn't find anything suitable.
So instead I set out to add this functionality to one myself.

I first looked at the source for [GNU make](https://www.gnu.org/software/make/)
however I couldn't understand the [source code](http://git.savannah.gnu.org/cgit/make.git/tree/)
so modifying that was out of the question. 
Instead I used `mk` 
as the base for my modifications. [Mk was originally developed
Plan 9 operating system](http://doc.cat-v.org/plan_9/4th_edition/papers/mk) 
as a re-write of make without many of its annoyances.
I had stumbled upon a simple [re-implementation of mk in Go by Daniel Jones](https://github.com/dcjones/mk)
a number of years ago and decided to add remote file support to that.
The changes I describe below are available on [my fork](https://github.com/ctSkennerton/mk)
of `mk` on Github.

The goal of my improvemnts was to make the following rule work:

```make
file.txt.gz: "https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/001/696/305/GCF_001696305.1_UCN72.1/GCF_001696305.1_UCN72.1_feature_count.txt.gz"
    curl $prereq > $target
```

That is, that `mk` would recognise the prerequisite as a URL,
determine if that URL was newer or older than the target and
proceed accordingly.
{{< sidenote "rule_quotes">}}  Since there is an extra colon 
in the URL we need to protect it with quotes so the mkfile parser 
doesn't get confused. {{< /sidenote >}}


The core of `make` and `mk` is deciding to remake a file based
on whether its prerequisites are newer than it. It does this by
looking at the last modified timestamp of a file. Sure enough inside
[`graph.go`](https://github.com/ctSkennerton/mk/blob/1e476df682360a522b993508429a13ae64d20685/graph.go#L64)
 there was a function `updateTimestamp` that gets the
last modified time of a file or sets that the file doesn't exist.

```go
info, err := os.Stat(u.name)
if err == nil {
	u.t = info.ModTime()
	u.exists = true
	u.flags |= nodeFlagProbable
} else {
	_, ok := err.(*os.PathError)
	if ok {
		u.t = time.Unix(0, 0)
		u.exists = false
	} else {
		mkError(err.Error())
	}
}
```

This was the function to modify to look at time stamps of remote
files. To do this we just need to identify files that look like
remote files, i.e. start with http(s):// or s3://. The following
simple modification makes that check and farms out the modification
checking based on if it's a http(s) or s3 remote file.

```go
// u is a node in the dependancy graph.
// its name member is the full path of the file
if strings.HasPrefix(u.name, "s3://") || strings.HasPrefix(u.name, "https://") || strings.HasPrefix(u.name, "http://") {
up, err := url.Parse(u.name)
if err != nil {
	log.Fatal(err)
}

if up.Scheme == "http" || up.Scheme == "https" {
	updateHttpTimestamp(u)
} else if up.Scheme == "s3" {
	updateS3Timestamp(u, up)
}
```

The implementation of `updateHttpTimestamp` is pretty simple.
A `HEAD` request is made to the URL and the `Last-Modified`
header is read. If that header is present the time is parsed
and used in the dependancy graph. If the header isn't found
it's assumed that the URL doesn't exist causing it to be remade.
```go
func updateHttpTimestamp(u *node) {
	// get the headers of the URL
	resp, err := http.Head(u.name)
	if err != nil {
		log.Fatal(err)
	}
	lastModified := resp.Header.Get("Last-Modified")
	if lastModified == "" {
		// no Last-Modified header so lets assume that it
		// doesn't exist
		u.t = time.Unix(0, 0)
		u.exists = false
	} else {
		tmptime, err := time.Parse(time.RFC1123, lastModified)
		if err != nil {
			log.Fatal(err)
		}
		u.t = tmptime
		u.exists = true
	}
}
```
 The implementation for updating an S3 file is similar but uses the 
 AWS API to get the last modified time.

 And with those small modifications the basic example I showed at the
 beginning now works. These modifications can be found in [my fork](https://github.com/ctSkennerton/mk)
 of `mk`, try it out and see how much easier your make-ing becomes!