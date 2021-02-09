# kong-plugin-oauth2

Sample use of kong oauth2 plugin

## Getting started

### Start Kong

Start kong in your local machine

Clone the repo: 

> https://github.com/Kong/docker-kong

Follow the intructions on

> https://github.com/Kong/docker-kong/tree/master/compose

### Configure Kong API

- create service

```sh
curl -iX POST 'localhost:8001/services' -H 'Content-Type: application/json' --data-raw '{
    "url": "http://viacep.com.br/ws/01001000/json/",
    "name": "viacep"
}'

HTTP/1.1 201 Created
Date: Tue, 09 Feb 2021 06:21:12 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Content-Length: 374
X-Kong-Admin-Latency: 12
Server: kong/2.2.1

{"host":"viacep.com.br","id":"bd1de683-cac9-48ce-a8dc-4b3e85d40c2c","protocol":"http","read_timeout":60000,"tls_verify_depth":null,"port":80,"updated_at":1612851672,"ca_certificates":null,"created_at":1612851672,"connect_timeout":60000,"write_timeout":60000,"name":"viacep","retries":5,"path":"\/ws\/01001000\/json\/","tls_verify":null,"tags":null,"client_certificate":null}
```

- create route

```sh
curl -iX POST 'http://localhost:8001/services/viacep/routes' -H 'Content-Type: application/json' --data-raw '{
    "paths": [
        "/viacep"
    ],
    "name": "viacep"
}'

HTTP/1.1 201 Created
Date: Tue, 09 Feb 2021 06:21:44 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Content-Length: 481
X-Kong-Admin-Latency: 18
Server: kong/2.2.1

{"id":"9803afe8-59ad-4221-a75f-a0c90f48ad05","tags":null,"paths":["\/viacep"],"destinations":null,"headers":null,"protocols":["http","https"],"strip_path":true,"created_at":1612851704,"request_buffering":true,"hosts":null,"name":"viacep","updated_at":1612851704,"snis":null,"preserve_host":false,"regex_priority":0,"methods":null,"sources":null,"response_buffering":true,"https_redirect_status_code":426,"path_handling":"v0","service":{"id":"bd1de683-cac9-48ce-a8dc-4b3e85d40c2c"}}
```

- test

```sh
curl -iX GET 'http://localhost:8000/viacep'

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Server: nginx/1.18.0
Date: Tue, 09 Feb 2021 06:22:04 GMT
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, OPTIONS
Access-Control-Allow-Headers: Content-Type, X-Request-With, X-Requested-By
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
Expires: Tue, 09 Feb 2021 07:22:04 GMT
Cache-Control: max-age=3600
Pragma: public
Cache-Control: public
X-Kong-Upstream-Latency: 672
X-Kong-Proxy-Latency: 19
Via: kong/2.2.1

{
  "cep": "01001-000",
  "logradouro": "Praça da Sé",
  "complemento": "lado ímpar",
  "bairro": "Sé",
  "localidade": "São Paulo",
  "uf": "SP",
  "ibge": "3550308",
  "gia": "1004",
  "ddd": "11",
  "siafi": "7107"
}
```

- add plugin on route

```sh
curl -iX POST http://localhost:8001/routes/viacep/plugins \
    --data "name=oauth2"  \
    --data "config.scopes=cep" \
    --data "config.mandatory_scope=true" \
    --data "config.enable_authorization_code=true" \
    --data "config.enable_client_credentials=true"
    
HTTP/1.1 201 Created
Date: Tue, 09 Feb 2021 06:22:45 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Content-Length: 708
X-Kong-Admin-Latency: 12
Server: kong/2.2.1

{"created_at":1612851765,"id":"7065e8ea-6255-45dc-bbd6-51375f936cfe","tags":null,"enabled":true,"protocols":["grpc","grpcs","http","https"],"name":"oauth2","consumer":null,"service":null,"route":{"id":"9803afe8-59ad-4221-a75f-a0c90f48ad05"},"config":{"pkce":"lax","accept_http_if_already_terminated":false,"reuse_refresh_token":false,"token_expiration":7200,"mandatory_scope":true,"enable_client_credentials":true,"hide_credentials":false,"enable_authorization_code":true,"enable_implicit_grant":false,"global_credentials":false,"refresh_token_ttl":1209600,"enable_password_grant":false,"scopes":["cep"],"anonymous":null,"provision_key":"D8zq9VsWqeJ9JBQaUd4oifQNJy0bhDRo","auth_header_name":"authorization"}}
```

- create consumer

```sh
curl -iX POST http://localhost:8001/consumers/ \
    --data "username=newapp" \
    --data "custom_id=newapp"
    
HTTP/1.1 201 Created
Date: Tue, 09 Feb 2021 06:23:16 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Content-Length: 122
X-Kong-Admin-Latency: 9
Server: kong/2.2.1

{"custom_id":"newapp","created_at":1612851796,"id":"57404e0d-3cf6-4012-8a01-adf7f7fa78ab","tags":null,"username":"newapp"}
```

- create oauth2 application

```sh
curl -iX POST http://localhost:8001/consumers/newapp/oauth2 \
    --data "name=newapp" \
    --data "client_id=newapp" \
    --data "client_secret=hardpassword" \
    --data "redirect_uris=http://localhost:8000" \
    --data "hash_secret=true"
    
HTTP/1.1 201 Created
Date: Tue, 09 Feb 2021 06:23:44 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Content-Length: 380
X-Kong-Admin-Latency: 31
Server: kong/2.2.1

{"created_at":1612851824,"id":"152d4154-c0af-48d3-a608-e463e434bc6c","tags":null,"name":"newapp","client_secret":"$pbkdf2-sha512$i=10000,l=32$lO8fBfbuiRNZuO2C214EpQ$32VoggOriD6UhfcFro5cRxFRJF9IW9kwAkB8xP1WIR4","client_id":"newapp","redirect_uris":["http:\/\/localhost:8000"],"hash_secret":true,"client_type":"confidential","consumer":{"id":"57404e0d-3cf6-4012-8a01-adf7f7fa78ab"}}
```

- test again

```sh
curl -iX GET 'http://localhost:8000/viacep'

HTTP/1.1 401 Unauthorized
Date: Tue, 09 Feb 2021 06:16:36 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
WWW-Authenticate: Bearer realm="service"
Content-Length: 77
X-Kong-Response-Latency: 1
Server: kong/2.2.1

{"error_description":"The access token is missing","error":"invalid_request"}
```

- create token

```sh
curl -ik -u newapp:hardpassword -X POST 'https://localhost:8443/viacep/oauth2/token' -H 'Content-Type: application/x-www-form-urlencoded' --data-urlencode 'grant_type=client_credentials' --data-urlencode 'scope=cep'

HTTP/2 200 
date: Tue, 09 Feb 2021 06:15:36 GMT
content-type: application/json; charset=utf-8
cache-control: no-store
pragma: no-cache
content-length: 91
x-kong-response-latency: 18
server: kong/2.2.1

{"token_type":"bearer","access_token":"N0udvR3zKPkHZO7oTJuCUjfqvtqNkBbj","expires_in":7200}
```

- call again using the token

```sh
# replace the token 3fe5jwRHkUSDPKyiagO2xA0tZJoZgKwD

curl -iX GET 'http://localhost:8000/viacep' -H 'Authorization: Bearer 3fe5jwRHkUSDPKyiagO2xA0tZJoZgKwD'

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Server: nginx/1.18.0
Date: Tue, 09 Feb 2021 06:15:56 GMT
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, OPTIONS
Access-Control-Allow-Headers: Content-Type, X-Request-With, X-Requested-By
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
Expires: Tue, 09 Feb 2021 07:15:56 GMT
Cache-Control: max-age=3600
Pragma: public
Cache-Control: public
X-Kong-Upstream-Latency: 345
X-Kong-Proxy-Latency: 2
Via: kong/2.2.1

{
  "cep": "01001-000",
  "logradouro": "Praça da Sé",
  "complemento": "lado ímpar",
  "bairro": "Sé",
  "localidade": "São Paulo",
  "uf": "SP",
  "ibge": "3550308",
  "gia": "1004",
  "ddd": "11",
  "siafi": "7107"
}
```
