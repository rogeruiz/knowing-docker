Linking Containers Together
---

Connecting containers together is [only one way][*] for Docker containers to
communicate with one another.

[*]: http://docs.docker.com/userguide/dockerlinks/

Exposing a port from a daemonized Docker container on 5000:

```sh
docker run -d -p 5000:5000 training/webapp python app.py
```

Now this is not the best way to bind a port because it restricts you to a single
exposed port for any given container. The `-p` flag by default binds the specified
port to all interfaces on the host machine. That functionality can be modified
by specifying a binding to a specific interface, like `localhost`:

```sh
docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```

Binding the 5000 port of the container to a dynamic port, but only on `localhost`
can be done with:

```sh
docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```

Checking the port bindings of a container can be done with the following command:

```sh
docker port [container_name] [port_number]
```

##### [The importance of naming your containers][*]

[*]: http://docs.docker.com/userguide/dockerlinks/#the-importance-of-naming

Docker will [automatically name your containers for you][*]. But it's sometimes
useful to name your containers yourself:

1. Naming your containers, like naming anything, helps you remember what you want
to use it for or what you expect it to do. For example: naming a container `web`
to know that it is the web server container.
2. Names provide a reference point to Docker to refer to other containers. You
can specify a link to the container `web` to container `db` to container `api`.

[*]: https://github.com/docker/docker/blob/master/pkg/namesgenerator/names-generator.go

> Note: Container names have to be unique. That means you can only call one
> container web. If you want to re-use a container name you must delete the old
> container (with docker rm) before you can create a new container with the same
> name. As an alternative you can use the --rm flag with the docker run command.
> This will delete the container immediately after it is stopped.

##### [Communication across links][*]

[*]: http://docs.docker.com/userguide/dockerlinks/#communication-across-links

Set up a link between containers using the `--link` flag on the `docker run`
command.

```sh
docker run -d --name db training/postgres
docker run -d -P --name web --link db:db training/webapp python app.py
```

Linking containers is so much more boss than actually using ports. While ports
may feel comfortable to me because that's all I've ever really worked with on
my own servers, linking has a big benefit where source container, `db`, is never
exposed to the network. This is because the link allows a source container to
provide information about itself to a recipient container. So the flag for linking
breaks down like so, `--link <source>:<alias>` with recipient being the `web`
container.

Docker provides this information in two ways:

1. Environment variables
2. Updating the `/etc/hosts` file

Environment variables that Docker generates are also included with the variables
originating from the `ENV` commands in the source container's `Dockerfile` and
the `-e`, `--env`, and `--env-file` options on the `docker run` command when the
source container is started. All environment variables originating from Docker
are available to any container that links to it. Sensitive data should not be
referenced in the environment variables passed to Docker. Keep those kinds of
things to yourself, or your app's self, rather than in `docker` commands or any
`Dockerfile` files.

The environment variables that Docker generates are broken down as follows:

* `<alias>_NAME`
	* When using the `--link db:webdb` flag, Docker creates a `WEBDB_NAME=/web/webdb`
	variables in the `web` container.
* `<name>_PORT_<port>_<protocol>` prefix format
	* The alias `<name>` specified in `--link`
	* The `port` number exposed
	* The `protocol` which is either TCP or UDP
* `<prefix>_ADDR`
	* The IP address from the URL
* `<prefix>_PORT`
	* The port number from the URL
* `<prefix>_PROTO`
	* The protocol from the URL
* `<alias>_PORT`
	* A URL of the container's first exposed port. TCP trumps UDP here.
* `<alias>_ENV_<name>`
	* Each environment variable passed into Docker creates one of these.

Unlike `/etc/hosts` entries, the environment variables IP addresses do not get
automatically updated if the source container is restarted. It is best to rely
on the kindness of host entries in `/etc/hosts` to resolve IP addresses of linked
containers. The following is the anatomy of an `/etc/hosts` in a source container:

```sh
172.17.0.7 <source_container_id>
...
172.17.0.5 <alias> <recipient_container_id> <recipient_container_name>
```

This means you can easily run a command like `ping` from inside a container using
the container's name / alias to communicate with it.

```sh
root <source_container_id>: apt-get install -yqq inetutils-ping
root <source_container_id>: ping <alias> # or <recipient_container_name>
```

> Note: You can link multiple recipient containers to a single source. For
> example, you could have multiple (differently named) web containers attached
> to your db container.
