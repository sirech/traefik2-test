# Traefik2 Test

This is intended to test [Traefik 2](https://traefik.io/) capabilities. It is a port from [this](https://github.com/sirech/traefik-test) repository.

## Requirements

- [Docker](https://www.docker.com/) 
- [docker-compose](https://docs.docker.com/compose/) >= 3

## Setup

### Enable domain

In order to test the echo app, you need to add the domain to the `/etc/hosts` file:

```bash
sudo sh -c "echo '127.0.0.1 echo.testing.com' >> /etc/hosts"
```

Generate a self signed certificate for the url with:

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout traefik/certs/cert.key -out traefik/certs/cert.crt
```

Generate a self signed client certificate for mTLS with:

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout traefik/certs/client.key -out traefik/certs/client.crt
```

### Running the code

_Traefik_ and the three example apps are run in different compose files:

```
docker-compose -f docker-compose.base.yml up --build
docker-compose -f docker-compose.app1.yml up
docker-compose -f docker-compose.app2.yml up
docker-compose -f docker-compose.app3.yml up
```

## Access

- The ui is available under `localhost:8080`
- The first app is reached through `localhost/findme`
- The second app is reached through `https://echo.testing.com/standard`
- The third app is reached through `https://echo.testing.com:4443/tls` and requires a client certificate. The certificate can be installed in `Postman` following [this guide](https://www.getpostman.com/docs/v6/postman/sending_api_requests/certificates)

## What is being tested

- *Discovery*: Traefik picks new applications automatically: When you start one of the example apps it appears in the ui automatically
- *Routing*: Traefik sends requests based on the configuration, which is passed through labels. No restart in `traefik` itself is needed for this
- *Password Access for the admin interface*: It is generated through `htpasswd`
- *Load Balancing*: Two instances of the same app receive traffic using `wrr`
- *SSL Termination*: Using a self signed certificate, the requests to the echo app are sent through http
- *Mutual TLS*: Using a self signed client certificate, the client is checked as well
- *Access*: There are different `entryPoints` defined, and not all of them should be available for each app. This can be defined in the config for each service
- *Middleware*: The new Traefik allows middleware to be inserted before a request is passed to a service, and after a request has processed by a service. We append a url to the requests sent to the first app
