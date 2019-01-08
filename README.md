# Kong with Keycloak!

Stack technology that must you have for this tutorial are this

 - Docker 
 - Keycloak
 - Kong comunity

# Start Up Kong
To start Up kong you must to follow this instructions, in the next page https://docs.konghq.com/install/docker/?_ga=2.247544835.587039642.1546895709-1028765720.1542656686

   
## Start Up Keycloak with docker compose

To Start up Keycloak you must execute this command 

    docker-compose -f keyclaok-compose.yaml up

##  x509 setup

Before setting up any tools we have to create a certificate. This certificate will be used by the IAM to sign the JSON Web Tokens (with the private key). And the certificate will be also used by the API Gateway to verify the tokens.

Here a quick (and dirty) method to generate a self signed certificate:


    $ # Generate the private key
    $ openssl genrsa -out mykey.pem 1024
    $ # Extract the public key
    $ openssl rsa -in mykey.pem -pubout -out mykey-pub.pem
    $ # Generate the certificate request
    $ openssl req -new -key mykey.pem -out mycert.csr -subj '/CN=my-common-name/'
    $ # Generate the certificate
    $ openssl req -x509 -sha256 -days 365 -key mykey.pem -in mycert.csr -out mycert.crt
    $ # Cleaning
    * rm mycert.csr

##  Kong setup

**Add service**

    curl -i -X POST \
      --url http://localhost:8001/services/ \
      --data 'name=example' \
      --data 'url=http://mockbin.org'
      
**Add route**

        curl -i -X POST \
      --url http://localhost:8001/services/example-two/routes \
      --data 'hosts[]=example-two.com'
     

Copy the route's id 

**Add JWT plugin to route**

    curl -X POST http://localhost:8001/routes/{id-route}/plugins \
        --data "name=jwt"
**Add Consumer** 

    curl -X POST http://localhost:8001/consumers \
        --data "username=web"
        
Copy the consumer's id

**Add JWT to the Consumer**

    curl -i -X POST http://localhost:8001/consumers/{id consuner}/jwt \
        -F "algorithm=RS256" \
        -F "rsa_public_key=@{path}/mykey-pub.pem" \
        -F "key={key is the realms url, with the host like this http://localhost:8084/auth/realms/{name of realms}}" # the `iss` field

## Keycloak Setup

 1. Create a client with secret
 2. Add user to realm
 3. import the collection  Kong.postman_collection.json
 4. replace the data
 5. Call the api an get the access_token
 6. copy the access_token and call the route 


## Call the route with JWT

    curl -i -X GET \
      --url http://localhost:8000/ \
      --header 'Host: example' \
      -H 'Authorization: Bearer {access_token}'


## Reference

https://ncarlier.gitbooks.io/oss-api-management/content/howto-kong_with_keycloak.html