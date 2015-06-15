Dockerizing Applications
---

[Running an applications in Docker takes a single command, `docker run`][*]

[*]: http://docs.docker.com/userguide/dockerizing/

The `docker run` command syntax is pretty easy to understand. It looks like so:

`docker run [image] [command]`

In the example below, I call `/bin/echo` to display 'Hello World'.

```sh
docker run ubuntu:14.04 /bin/echo 'Hello World'
```

In the example below, I call `/bin/bash` which will login me into the bash shell within
the machine. The extra flags `-t` and `-i` will each do a different thing.

* `-t` will start a terminal session within the container
* `-i` will create an interactive connection which will display all the `STDIN`
from the container

```sh
docker run -t -i ubuntu:14.04 /bin/bash
```

Now, running a container in your terminal session is cool. But, you really are
going to be daemonizing your containers so that you can get on with your life
while the container does what it does best. Contain the processes of your application.

In the example below, I call `/bin/sh` from the machine passing in the `-c` flag.
The `-c` flag will call the follow string as commands to `/bin/sh` within the
container. The `-d` flag will daemonize my container and allow me to control my
session again while my docker container keeps outputting 'hello world' every one
second or so.

```sh
docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

You can see the logs of a container with the `docker logs` command passing in
the container id you can see when you run `docker ps`. The container id is a
[256-bit hex-encoded unique identifier][*].

[*]: https://news.ycombinator.com/item?id=9442411
