# Running TailsX with Docker

## Table of contents
- [Running TailsX with Docker](#running-tailsx-with-docker)
  - [Table of contents](#table-of-contents)
- [Modify the Dockerfile (aptly named `Dockerfile`)](#modify-the-dockerfile-aptly-named-dockerfile)
  - [Add your domain](#add-your-domain)
  - [Configuring Gunicorn (Optional)](#configuring-gunicorn-optional)
- [Build \& run TailsX](#build--run-tailsx)
- [Test your server](#test-your-server)

TailsX can be ran under a Docker container easily.

# Modify the Dockerfile (aptly named `Dockerfile`)
## Add your domain
There should be a line in the Dockerfile that looks like this;
```dockerfile
# ENV DOMAIN=https://www.yourdomain.com
```
Uncomment this line by removing th `#` and change the domain to your own domain.

## Configuring Gunicorn (Optional)
The default configuration of Gunicorn works, but you can optionally configure it to your liking.

The command that will be called can be found in `CMD` (very end of the file). The arguments are seperated into different strings in an array.

See [the gunicorn docs](https://docs.gunicorn.org/en/latest/run.html) on running gunicorn and [the Docker docs](https://docs.docker.com/engine/reference/builder/#cmd) on how the `CMD` instruction works.

# Build & run TailsX
Build and image and tag it with `tailsx:latest`
```bash
docker build -t tailsx:latest .
```

Run the image with docker-compose
```bash
docker-compose up
```

<i>NOTE: If you experience any permissions errors, you can try running both commands with `sudo`.</i>

# Test your server
See [here](/README.md#test-your-server) on testing your server.
