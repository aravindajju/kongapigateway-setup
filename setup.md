### Installing Kong API Gateway

Installation of Kong API gateway, requies installation of 
 + Kong API Gateway software
 + Setup and Run a database (PostgreSQL)

1. Pull Kong Gateway Docker Image and verify if it is installed.  Create Docker network 

```
docker pull kong-docker-kong-gateway-docker.bintray.io/kong-enterprise-edition:2.3.3.0-alpine

docker images

docker tag <IMAGE_ID> kong-ee

docker network create kong-ee-net
```  
2. Setup and run PostgreSQL

```
docker run -d --name kong-ee-database \
--network=kong-ee-net \
-p 5432:5432 \
-e "POSTGRES_USER=kong" \
-e "POSTGRES_DB=kong" \
-e "POSTGRES_PASSWORD=kong" \
postgres:9.6

docker run --rm --network=kong-ee-net \
-e "KONG_DATABASE=postgres" \
-e "KONG_PG_HOST=kong-ee-database" \
-e "KONG_PG_PASSWORD=kong" \
-e "KONG_PASSWORD=<SOMETHING-YOU-KNOW>" \
kong-ee kong migrations bootstrap
  
```
3. Start the Gateway

```
docker run -d --name kong-ee --network=kong-ee-net \
  -e "KONG_DATABASE=postgres" \
  -e "KONG_PG_HOST=kong-ee-database" \
  -e "KONG_PG_PASSWORD=kong" \
  -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
  -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
  -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
  -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
  -e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
  -e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
  -p 8000:8000 \
  -p 8443:8443 \
  -p 8001:8001 \
  -p 8444:8444 \
  -p 8002:8002 \
  -p 8445:8445 \
  -p 8003:8003 \
  -p 8004:8004 \
  kong-ee
  
```

4. Verify if the installation is working fine or not

```
curl -i -X GET --url http://localhost:8001/services

http://localhost:8002

```
**We are all set**
