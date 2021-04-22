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
-e "KONG_PASSWORD=acmdeccan123" \
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

### Setup an API

Setting up an API would require setting up of a Service (proxy on top of an actual implementation) and a Route to this service. Let us use an existing mocking service from mockbin.org

1. Create a service called 'mock_service'
```
curl -i -X POST http://localhost:8001/services \
  --data name=mock_service \
  --data url='http://mockbin.org'
```
2. Create a route

```
curl -i -X POST http://localhost:8001/services/mock_service/routes \
  --data 'paths[]=/mock' \
  --data name=mocking
```
3. Verify the APIs

```
curl -i -X GET http://localhost:8000/mock/request
curl -i -X GET http://mockbin.org/request
```
### Securing the API

Next step is to secure the API with a token.  So only authorized clients can use the API

1. Enable key based authentication plugin on 'mocking' route
```
curl -X POST http://localhost:8001/routes/mocking/plugins \
  --data name=key-auth


curl -i http://localhost:8000/mock
```
2. Once key based authentication is enabled, we need to create a consumer and assign a key

```
curl -i -X POST http://localhost:8001/consumers/ \
  --data username=consumer \
  --data custom_id=consumer
  
curl -i -X POST http://localhost:8001/consumers/consumer/key-auth \
  --data key=acmdeccan
  
```
3. Let us now test the API again
```
curl -i http://localhost:8000/mock/request \
  -H 'apikey:acmdeccan'
```

### Rate Limit service

Rate limiting is a mechanism to ensure APIs are used fairly as per the quotas. 

1. Enable rate limiting plug in with 5 requests a minute

```
curl -i -X POST http://localhost:8001/plugins \
  --data name=rate-limiting \
  --data config.minute=5 \
  --data config.policy=local
```
2. Test it 6 times  
```  
curl -i -X GET http://localhost:8000/mock/request

```  







  




