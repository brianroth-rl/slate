# Authentication

During initial setup the Reach Local API will make a GET request to the registered redirect_uri address with a code parameter.  This code is the authorization token to be used during the REQUEST ACCESS TOKEN section.

Initial setup is not possible for the QA environment as the login credentials cannot be disseminated to outside vendors.  A refresh token has been provided that can be used with the refresh access token instructions found below to obtain an access token.

## Initial Setup

This needs to be performed once for each environment or if latest refresh token if somehow lost. This process will allow you to retrieve an authorization token which can be used to request an access token and refresh token.

In a browser go to 

`https://externalapi.reachlocal.com/oauth/authorize?client_id=[CLIENT_ID]&response_type=code&redirect_uri=[REDIRECT_URI]`

Where CLIENT_ID is provided during implementation.  REDIRECT_URI is a URL provided by the client at implementation time and stored with the OAuth provided.

If done correctly you’ll see a login screen. Enter the credentials that were provided at account setup time.  These are the same credentials used to authenticate with the Edge application.

![login screen](/images/login.png)

Upon successful authentication, you’ll be shown TOS which you must agree to. Then you’ll be redirected to the REDIRECT_URI along with an authorization token that expires in 10 minutes. If you fail to use it before it expires then you must repeat these steps again.

Use the authorization token to request an access and refresh token.

## Requesting Access and Refresh Tokens
```ruby
require 'uri'
require 'net/http'

url = URI("https://externalapi.reachlocal.com/oauth/token")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true
http.verify_mode = OpenSSL::SSL::VERIFY_NONE

request = Net::HTTP::Post.new(url)
request["content-type"] = 'application/json'
request.body = "{\n\t\"client_id\": \"CLIENT_ID\",\n\t\"client_secret\": \"CLIENT_SECRET\",\n\t\"code\": \"AUTHORIZATION_TOKEN\",\n\t\"grant_type\": \"authorization_code\",\n\t\"redirect_uri\": \"REDIRECT_URI\"\n}"

response = http.request(request)
puts response.read_body
```

```java
OkHttpClient client = new OkHttpClient();

MediaType mediaType = MediaType.parse("application/json");
RequestBody body = RequestBody.create(mediaType, "{\n\t\"client_id\": \"CLIENT_ID\",\n\t\"client_secret\": \"CLIENT_SECRET\",\n\t\"code\": \"AUTHORIZATION_TOKEN\",\n\t\"grant_type\": \"authorization_code\",\n\t\"redirect_uri\": \"REDIRECT_URI\"\n}");
Request request = new Request.Builder()
  .url("https://externalapi.reachlocal.com/oauth/token")
  .post(body)
  .addHeader("content-type", "application/json")
  .build();

Response response = client.newCall(request).execute();
```

```shell
curl -X POST \
  https://externalapi.reachlocal.com/oauth/token \
  -H 'content-type: application/json' \
  -d '{
	"client_id": "CLIENT_ID",
	"client_secret": "CLIENT_SECRET",
	"code": "AUTHORIZATION_TOKEN",
	"grant_type": "authorization_code",
	"redirect_uri": "REDIRECT_URI"
}'
```
> The above command returns JSON structured like this:

```json
{
  "access_token": "8821d2a47a88d1cbaced1043aee73ccc4608d7cacb068c0eb09e468e91cdb49b",
  "token_type": "bearer",
  "expires_in": 7200,
  "refresh_token": "5d59deea16599eb316a63989a9974568da4f5570e7ef9969c693f0b1ce1052cb",
  "created_at": 1495549720
}

```

You’ll use the authorization token from above to request an initial access token and refresh token. The access token is what you’ll pass in the authorization header for any report API request. The refresh token is what you will use to request a new access token when it expires. It is very important that you save the refresh token otherwise you won’t be able to request a new access token. If this ever happens then you need to repeat the initial setup. NOTE: When you request / refresh an access token you’ll be provided with a new refresh token.

## Refresh Access Token
```ruby
require 'uri'
require 'net/http'

url = URI("https://externalapi.reachlocal.com/oauth/token")

http = Net::HTTP.new(url.host, url.port)
http.use_ssl = true
http.verify_mode = OpenSSL::SSL::VERIFY_NONE

request = Net::HTTP::Post.new(url)
request["content-type"] = 'application/json'
request.body = "{\n\t\"client_id\": \"CLIENT_ID\",\n\t\"client_secret\": \"CLIENT_SECRET\",\n\t\"grant_type\": \"refresh_token\",\n\t\"refresh_token\": \"REFRESH_TOKEN\"\n}"

response = http.request(request)
puts response.read_body
```

```java
OkHttpClient client = new OkHttpClient();

MediaType mediaType = MediaType.parse("application/json");
RequestBody body = RequestBody.create(mediaType, "{\n\t\"client_id\": \"CLIENT_ID\",\n\t\"client_secret\": \"CLIENT_SECRET\",\n\t\"grant_type\": \"refresh_token\",\n\t\"refresh_token\": \"REFRESH_TOKEN\"\n}");
Request request = new Request.Builder()
  .url("https://externalapi.reachlocal.com/oauth/token")
  .post(body)
  .addHeader("content-type", "application/json")
  .build();

Response response = client.newCall(request).execute();
```

```shell
curl --request POST \
  --url https://externalapi.qa.reachlocal.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{\n	"client_id": "CLIENT_ID",\n	"client_secret": "CLIENT_SECRET",\n	"grant_type": "refresh_token",\n	"refresh_token": "REFRESH_TOKEN"\n}'
}'
```
> The above command returns JSON structured like this:

```json
{
    "access_token": "62b28b310f73caa88d6968c380ccb85434f4f6379162e01d1bade4b62a62762c",
    "token_type": "bearer",
    "expires_in": 7200,
    "refresh_token": "f2b8a81ca929a4df4d40fe8bb36a379761f91242ac3b26957e3e0a6696235233",
    "created_at": 1504822139
}

```

Every API request must include an access token in the authorization header. This access token expires every 2 hours. When the token expires you’ll receive a 401 HTTP status code. You can request a new access token by using the refresh token you were given with your previous access token. It is very important that you save the refresh token otherwise you won’t be able to request a new access token. If this ever happens then you need to repeat the initial setup.


## Security Considerations

Refresh Tokens are long-lived. This means when a client gets one from a server, this token must be stored securely to keep it from being used by potential attackers, for this reason, it is not safe to store them in the browser. If a Refresh Token is leaked, it may be used to obtain new Access Tokens (and access protected resources) until it is blacklisted. Refresh Tokens must be issued to a single authenticated client to prevent the use of leaked tokens by other parties. Access Tokens must also be kept secret, but due to its shorter life, security considerations are less critical.

