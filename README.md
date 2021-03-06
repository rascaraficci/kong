# kong

## Kong with custom plugin

The Docker image generated by the Dockerfile contains a kong custom plugin that extract roles from a JWT token and make a request for a Policy Decision Point (PDP)

An example of how to associate a previously created service to the plugin:

``` sh
kong="http://kong:8001"
service_name="service_example"
url_pdp="http://auth:5000/pdp"

curl -X POST \
--url ${kong}/services/${service_name}/plugins/ \
--data "name=pepkong" \
--data "config.pdpUrl=${url_pdp}"
```

The configuration below shows how the image can be used with docker-compose:

``` yml
version: '3.7'
services:

  db:
    image: dojot/postgres:9.5.21-alpine
    environment:
      POSTGRES_DB: kong
      POSTGRES_USER: kong
      POSTGRES_PASSWORD: kong
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "kong"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: on-failure

  kong:
      image:  kong_dojot
      user: "kong"
      depends_on:
        - db
      environment:
        KONG_ADMIN_ACCESS_LOG: /dev/stdout
        KONG_ADMIN_ERROR_LOG: /dev/stderr
        KONG_ADMIN_LISTEN: '0.0.0.0:8001'
        KONG_CASSANDRA_CONTACT_POINTS: postgres
        KONG_DATABASE: postgres
        KONG_PG_HOST: db
        KONG_PG_USER: kong
        KONG_PG_DATABASE: kong
        KONG_PG_PASSWORD: kong
        KONG_PROXY_ACCESS_LOG: /dev/stdout
        KONG_PROXY_ERROR_LOG: /dev/stderr
        KONG_LOG_LEVEL: info
      ports:
        - "8000:8000/tcp"
        - "127.0.0.1:8001:8001/tcp"
        - "8443:8443/tcp"
        - "127.0.0.1:8444:8444/tcp"
      healthcheck:
        test: ["CMD", "kong", "health"]
        interval: 10s
        timeout: 10s
        retries: 10
      restart: on-failure
```
