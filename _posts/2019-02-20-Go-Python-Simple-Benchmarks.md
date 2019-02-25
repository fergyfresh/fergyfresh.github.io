---
layout: post
title: Benchmarking Go and Python
---

# Simple Benchmarking for GET requests between Gin and Flask

## Preamble

You need to have Go installed and Python installed for this to work.

## Introduction

Just noticed after finishing up all the code for this post that both of these are drinking related (Gin and Flask). After analyzing the go ecosystem for web frameworks (Gin, Martini, Web.go, Beego, Goji, Gorilla) I decided on pitting Gin against Flask. This appeared to be the most straight forward comparison between what I'm comfortable with as one of the more popular python web frameworks, but also it's known as one of the fastest ones of the bunch.

Ideally I would do this in a few files, and probably use gunicorn in production for Python and structure the app differently, but I just wanna build the most stripped down version of this example in ython to compare pure speeds of Flask vs. Gin and the size of their minimal docker builds for just returning a simple JSON object on a get request to the app.

Make a directory called pygo-benchmark and change your current directory to that dir.

## Go web-app with Gin

Do this all in a subdirectory within pygo-benchmark, I called it go-stuff, but you can call it whatever you'd like.

### Write the file

Lets get to building. I use vim-go so you get a whole package scaffold upon the first `vim file.go` command. So when I type `vim main.go` I get:

```go
package main

import "fmt"

func main() {
	fmt.Println("vim-go")
}
```

So you should also create a file called `main.go` and add the code above to that file. Now we are going to want to install gin. In order to do this we are going to type `go get -u github.com/gin-gonic/gin` and that will install the go module for us. Alright, now that we have gin installed lets modify the file to make our `main.go` use gin now and return a simple json object on a GET to `localhost:8080`, the default port for this webserver.

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	router := gin.Default()

	router.GET("/", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})

	// router runs on :8080 by default
	router.Run()
}

```

### Build and Run

It is pretty simple to build a file with go, `go build main.go` is all you need. Now you should have an executable file called main in the same directory you're in. If you just run `/main` in the command line at the same directory you built the file from, it should have some standard output like:

```
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /                         --> main.main.func1 (3 handlers)
[GIN-debug] Listening and serving HTTP on :8080
```

If you've gotten this far congrats you built and ran your first go app!

## Python web-app with Flask

Do all of this in a subdirectory in pygo-benchmark, I called it flask-stuff, but once again do whatever you need to here.

### Write the file

In order to make this post shorter I'm not going to go through the whole setup, I'm just gonna *blurgghhhhh* the file out here. Make a file called `main.py` with the contents below:

```python
from flask import Flask

app = Flask(__name__)


@app.route("/")
def ping():
    return '{"message": "ping"}'

if __name__ == "__main__":
    app.run(host="0.0.0.0")
```

## Build and Run python

So there is no build in python, we will get to that benefit as one the favors Go in our Benchmarking section.

I like to do everything on the commandline in virtualenv, but if you dont mind messing up your packages do whatever you want, this is what I would do in order to get this file running:

```
virtualenv .venv -p python3.6
source .venv/bin/activate
pip install flask
python main.py
```

Because of the way this file is setup with the `if __name__...` stuff we can just run this file from the command line, like this `python main.py` and we should see something like this if we did it successfully:

```
(.venv) ➜  flask-stuff ✗ python main.py
 * Serving Flask app "main" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 313-728-815
 ```

## Benchmarking the pure speed of each setup.

### In python using requests and timeit

Now let's get the standard boilerplate dislaimer out here. You know, the one about how results may vary on the machine that you run the benchmarks on and that you really need to have a multitude of machines in order to really prove out benchmars, yada-yada.

Alright, so with that boring stuff out of the way I LOVEEE using `timeit` to time stuff in python. You can do multiple runs one after the other in order to find a more valuable benchmark (the average response time).

Since we should have both webservers running as of now in this blog the go server is running on `localhost:8080` and the python on at `localhost:5000`. Lets time them both, shall we?:

```
pip install requests
python
>>>
>>>import timeit
>>>import requests
>>>
>>> # gin / go
>>> timeit.timeit("requests.get('http://localhost:8080')", setup="import requests", number=10000)
21.956489593023434
>>> # flask / python
>>> timeit.timeit("requests.get('http://localhost:5000')", setup="import requests", number=10000)
34.483878459897824
```

Cool. So as you can see the the Go option is ~36% faster! That's amazing, no wonder people are going for go > python these days. But then again this isn't the most fair test as we can put a multithreadable WSGI server in front of our python setup. Now onto round two...

### Using a new fangled benchmarking tool called wrk

 and multithreading all the things! Thanks to a great suggestion by a dedicated reader who tweeted in, I am also adding in a multhreaded benchmark and adding gunicorn to sit in front of our Flask app.
 
 In order to get up to speed with me you'll have to download and make the binary for wrk using the instructions found [here](https://github.com/wg/wrk/wiki/Installing-wrk-on-Linux) and if you look there are more options on the side if you're a Windows/Mac user. Also we are going to wanna `pip install gunicorn` as well and in the flask-stuff directory we are going to want to type `gunicorn -w 8 main:app` which tells gunicorn (our multithreaded WSGI server) to use 8 threads (since the output of `nproc --all` on my machine was 8) and it says look at the `main.py` file and the `app` function as the entrypoint for the app.
 
 Now down to benchmarking the two using "a real benchmarking tool" - Some guy named Ben.
 

```
➜  go-hello wrk -t8 -c32 -d30s http://127.0.0.1:5000 # flask-python
Running 30s test @ http://127.0.0.1:5000
  8 threads and 32 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.58ms    1.70ms  42.16ms   94.75%
    Req/Sec     1.12k   158.54     2.83k    84.48%
  268403 requests in 30.10s, 45.82MB read
Requests/sec:   8917.30
Transfer/sec:      1.52MB
➜  go-hello wrk -t8 -c32 -d30s http://127.0.0.1:8080 # go-gin
Running 30s test @ http://127.0.0.1:8080
  8 threads and 32 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.64ms    4.71ms  60.39ms   85.07%
    Req/Sec     6.86k     0.93k   33.27k    82.09%
  1639477 requests in 30.10s, 220.46MB read
Requests/sec:  54471.39
Transfer/sec:      7.32MB
```

Okay, maybe ben was right, this really shows how much better go is than python. Surprisingly though, the one thing python did have was a more deterministic response time as the stdev of the latency was < 2ms where it was almost 5ms in go.

But once again, this is another benchmark proving out go is faster than python. Req/sec wise go is 6.125 times faster than flask, on average.

## Benchmarking for Docker image size.

Alrighty then. I'm just gonna go ahead and say it that this post is TOO freaking long already and you're just not going to get me to breakdown the multistage docker builds I iterated through in order to get the smallest image size for each one.

### Python Dockerfile

Lets put this file in the `flask-stuff/Dockerfile` directory with the contents below:

Our structure should look like this:

- flask-stuff
  * main.py
  * Dockerfile

```docker
FROM python:3.6-alpine
COPY . /
WORKDIR /
RUN pip install flask
RUN pip install gunicorn
CMD ["gunicorn"  , "-w", "8", "main:app"]
```

In order to build this you'd type `docker build . -t benchmark/py`.

### Go Dockerfile

Lets put this file in the `go-stuff/Dockerfile` directory with the contents below:

Our structure should look like this:

- go-stuff
  * main.go
  * Dockerfile

```docker
FROM golang:latest
ADD . .
RUN go get -u github.com/gin-gonic/gin
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .
RUN ls
RUN pwd

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
COPY --from=0 /go/main .
CMD ["./main"] 
```

In order to build this you'd type `docker build . -t benchmark/go`.

### Surely what are the results?!?

I have the results, and my name's not Shirley! Bad joke, I know, but I'm trying to keep you awake here.

Alright so here's the data:
```
➜  pygo-benchmark docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
benchmark/go        latest              02ef15405471        About an hour ago   21.7MB
benchmark/py        latest              8fd24d5e54f1        About an hour ago   89.6MB
python              3.6-alpine          83d065b0546b        19 hours ago        79MB
golang              latest              901414995ecd        2 weeks ago         816MB
alpine              latest              caf27325b298        3 weeks ago         5.53MB
```

Now this is astonishing! The `benchmark/go` image is ~76% smaller than the `benchmark/py` image. This is because of a few reasons, go literally just needs that one binary `main` to run, where as python needs all of the supporting libraries because it isn't compiled.

If we used the golang image as a base instead of the alpine image as a base for our `benchmark/go` image it would be ~835MB. The reason why Go docker images are so small is because you can run them directly on an alpine image, which is ~5MB.

# Questions, comments, concerns

Feel free to click one of these buttons in order to signal me with something that was messed up with this. I'd be glad to fix anything that didnt work for you.
