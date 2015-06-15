Running a web application in Docker
---

Using a pre-built image from [Docker Hub][*], the following command will run a
Python Flask application.

[*]: https://registry.hub.docker.com/search?q=library

```sh
docker run -d -P training/webapp python app.py
```

The `-d` flag daemonizes our container and the `-P` flag will map any required
network ports from the container to our host machine. That's how we'll view our
application.[*][*]

[*]: http://docs.docker.com/userguide/usingdocker/#running-a-web-application-in-docker
