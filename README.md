nginx-le-proxy is a simple project to run the [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) container along with the [let's encrypt sidecar](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion) container.

### motivation

When you are using docker to run a web server, you will often bind that server to the host's port 80 and 443. That works just find if you only have one web server you are running. If you are running say 2 or more web servers, you run into port binding problems. You one approach is to change the host port the other web server containers bind to. This can be a big pain and can cause problems when working on a project with others that don't share your port choices. The approach with nginx-proxy is to have a proxy container that is the only container that binds to your host port 80 and 443. Then it will proxy requests to any other container based on the domain in the request. This allows you to have many web servers running in docker containers all accessible through the standard web ports of 80 and 443. The reason for this project is two fold:

1. Be a stand-alone project as to not be tied in with any specific project.
2. Automatically provide SSL/TLS certificates through [Let's Encrypt](https://letsencrypt.org/).

Even if you don't have the SSL need, it might be useful to avoid the binding to port 80 problem.

### installation

The installation is a simple as cloning the repo and spinning it up. You will need to make sure that:

1. docker and docker-compose are installed
2. there are no other ports or processes binding to port 80 and/or 443
3. an email to use with let's encrypt in a `.env` file

```shell
echo 'DEFAULT_EMAIL="foo@example.com"' > .env
```

### usage

To start it you can just bring it up with docker-compose.

```shell
docker-compose up -d
```

Now it is ready and waiting for any other containers to spin up with a VIRTUAL_HOST (and LETSENCRYPT_HOST for SSL/TLS) environment variable(s). In order for those other containers to be accessible to this nginx-proxy container, they will need to join the network created by this docker-compose file.

Inside your project's docker-compose file join the network like this:
```yaml
version: '3'
networks:
  nginx-le-proxy_net:
    external: true

services:
  my_web_server:
    ...
    environment:
      VIRTUAL_HOST: app.my-host.com     # must resolve to your host
      LETSENCRYPT_HOST: app.my-host.com # if you want an SSL cert for the domain
    ...
    networks:
      - default # still connect to the default network
      - nginx-le-proxy_net # connect to the proxy network so nginx-proxy can reach
```

See the [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) readme for more details.
