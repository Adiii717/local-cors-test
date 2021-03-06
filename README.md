# local-cors-test
Test CORS locally with curl and HTML &amp; Ajax

## Browser base test

You can update API in `ajax-api-call.js`
```js
var API="https://api.github.com"
```
#### CORS success response

if the cors working you will see response in the browser.

![Browser sucess response](https://github.com/Adiii717/local-cors-test/blob/main/images/api-response.png)

#### CORS failure case

if the backend has does not allow the origin it should block the request
![Browser failure case](https://github.com/Adiii717/local-cors-test/blob/main/images/cors-failure-case.jpg)

## Curl core test

```shell
curl -I -H "Origin: https://frontend.domain"  https://origintest.mydomain.com/
```

Expected allow origin header to check

```shell
access-control-allow-origin
```

Curl response:

```shell
curl -I -H "Origin: https://frontend.domain"  https://origintest.mydomain.com/
HTTP/2 200
date: Tue, 28 Sep 2021 04:47:05 GMT
content-type: text/html; charset=utf-8
content-length: 170
x-powered-by: Express
access-control-allow-origin: https://frontend.domain
access-control-allow-headers: X-Requested-With
etag: W/"aa-z+ebXSEdArbZ+EXlN/WQjf6HV8c"
strict-transport-security: max-age=15724800; includeSubDomains
access-control-allow-credentials: true
```

### Express setting for setting allow origin base on incoming request origin

```js
app.all('/*', function(req, res, next) {
  res.header("Access-Control-Allow-Origin", req.get('origin'));
  res.header("Access-Control-Allow-Headers", "X-Requested-With");
  next();
});
```

### Kubernetes ingress setting for setting allow origin base on incoming request origin

```yaml
ingressExternal:
  enabled: false
  annotations: 
     kubernetes.io/ingress.class: nginx-layer-tls
     nginx.ingress.kubernetes.io/ssl-redirect: "true"
     nginx.ingress.kubernetes.io/enable-cors: "true"
     nginx.ingress.kubernetes.io/cors-allow-methods: "PUT, GET, POST, OPTIONS, DELETE"
     nginx.ingress.kubernetes.io/cors-allow-credentials: "true"
     nginx.ingress.kubernetes.io/configuration-snippet: |
        more_set_headers "Access-Control-Allow-Origin: $http_origin";
        proxy_set_header l5d-dst-override $service_name.$namespace.svc.cluster.local:$service_port;
        grpc_set_header l5d-dst-override $service_name.$namespace.svc.cluster.local:$service_port;
     kubernetes.io/tls-acme: "true"
  paths:
    - "/"
  hosts:
    - mydomain.com
  tls:
    secretName: my-tls-secret
    hosts:
      - mydomain.com
```


### Kubernetes ingress setting for setting allow origin from same 

In the below case the kubernetes ingress will only request from same origin

```shell
curl -I -H "Origin: https://test.domain"  https://mydomain.com:443
HTTP/2 200
date: Tue, 28 Sep 2021 05:56:26 GMT
content-type: text/html; charset=utf-8
content-length: 7
x-powered-by: Express
etag: W/"7-3hjM+hXoQcTv5niKCOzNqTVlC7E"
strict-transport-security: max-age=15724800; includeSubDomains
access-control-allow-origin: https://mydomain.com:443 ## here you can only allowed domain and in this browser will throw error
access-control-allow-credentials: true
```

```yaml
ingressExternal:
  enabled: false
  annotations: 
     kubernetes.io/ingress.class: nginx-layer-tls
     nginx.ingress.kubernetes.io/ssl-redirect: "true"
     nginx.ingress.kubernetes.io/enable-cors: "true"
     nginx.ingress.kubernetes.io/cors-allow-methods: "PUT, GET, POST, OPTIONS, DELETE"
     nginx.ingress.kubernetes.io/cors-allow-credentials: "true"
     nginx.ingress.kubernetes.io/cors-allow-origin: "https://mydomain.com:443"
     nginx.ingress.kubernetes.io/configuration-snippet: |
        proxy_set_header l5d-dst-override $service_name.$namespace.svc.cluster.local:$service_port;
        grpc_set_header l5d-dst-override $service_name.$namespace.svc.cluster.local:$service_port;
     kubernetes.io/tls-acme: "true"
  paths:
    - "/"
  hosts:
    - mydomain.com
  tls:
    secretName: my-tls-secret
    hosts:
      - mydomain.com
```