# Signed JSON Web Tokens (JWT) implementation for PHP 7

## JWT

Read more about JWT here:

* [RFC 7519](https://tools.ietf.org/html/rfc7519)
* [jwt.io](https://jwt.io/introduction/)
* [JWT Handbook](https://auth0.com/resources/ebooks/jwt-handbook)

## License

Please check [BSD-3 Clause](http://opensource.org/licenses/BSD-3-Clause) terms before use.

## Supported algorithms

* HS256
* HS384
* HS512
* RS256
* RS384
* RS512

## Installation

```
composer require nowakowskir/php-jwt
```

## Elements

When using this package, you will be mostly interested in two classes: ```TokenEncoded``` and ```TokenDecoded```.

### TokenDecoded

This class is representation of decoded token. It consist of header and payload. If you want to create encoded token, you just need to prepare decoded version of token first and then use ```$tokenDecoded->encode($key)``` method. In result, you will get new object of ```TokenEncoded``` class.

### TokenEncoded

This class is representation of encoded token. You can decode token using ```$tokenEncoded->decode()```. In result, you will get new object of ```TokenDecoded``` class.

> Please note, providing key is not required to decode token, as its header and payload are public in signed JWT Tokens. You should take care not to pass any confidental information within token's header and payload. JWT only allows you to verify, if the token containing given payload was
issued by trusted party. It does not protect your data passed in payload!

You should use ```$tokenEncoded->decode()``` method if you need to access token's header or payload only.

In order to validate token you should use ```$tokenEncoded->validate($key)``` method.

## Usage

### Creating new token

```
$tokenDecoded = new TokenDecoded($header, $payload);
$tokenEncoded = $tokenDecoded->encode($key);

echo 'Your token is: ' . $tokenEncoded->__toString();
```

### Validating existing token


```
$tokenEncoded = new TokenEncoded('eyJhbGciOiJI...2StJdy+4XC3kM=');
        
try {
  $tokenEncoded->validate($key);
} catch (IntegrityViolationException $e) {
  // Token is not trusted
} catch(TokenExpiredException $e) {
  // Token expired (exp date reached)
} catch(TokenInactiveException $e) {
  // Token is not yet active (nbf or iat dates not reached)
} catch(Exception $e) {
  // Something else gone wrong
}
```

### Getting payload of existing token

```
$tokenEncoded = new TokenEncoded('eyJhbGciOiJI...2StJdy+4XC3kM=');
var_dump($tokenEncoded->decode()->getPayload());
```

> Please note, providing key is not required to decode token, as its header and payload are public in signed JWT Tokens. You should take care not to pass any confidental information within token's header and payload. JWT only allows you to verify, if the token containing given payload was
issued by trusted party. It does not protect your data passed in payload!

### Creating new token with custom algorithm

By default ```HS256``` algorithm is used to encode tokens. You can change algorithm by either providing it under ```alg``` key in token's header or as a parameter to ```encode()``` method. Because it's possible to provide algorithm in two ways, algorithm defined in token's header takes priority if provided in both places.

```
$tokenDecoded = new TokenDecoded(['alg' => JWT::ALGORITHM_HS384], $payload);
$tokenEncoded = $tokenDecoded->encode($key);
```

```
$tokenDecoded = new TokenDecoded([], $payload);
$tokenEncoded = $tokenDecoded->encode($key, JWT::ALGORITHM_HS384);
```

```
$tokenDecoded = new TokenDecoded(['alg' => JWT::ALGORITHM_HS384], $payload);
$tokenEncoded = $tokenDecoded->encode($key, JWT::ALGORITHM_HS512);
// HS384 algorithm will take priority
```

Please note there is no need to provide algorithm when validating token as algorithm is already contained in token's header.

```
$tokenEncoded = new TokenEncoded($tokenString);
$tokenEncoded->validate($key);
```

### Using private/public key pair to sign and validate token

First you need to generate private key.

```
ssh-keygen -t rsa -b 4096 -m PEM -f private.key
```

Next, you need to generate public key based on private key.

```
openssl rsa -in private.key -pubout -outform PEM -out public.pub
```

Private key will be used to sign a token whereas public key will be used to verify a token.

```
$tokenDecoded = new TokenDecoded();
$privateKey = file_get_contents('./private.key');
$tokenEncoded = $tokenDecoded->encode($privateKey, JWT::ALGORITHM_RS256);

$tokenString = $tokenEncoded->__toString();
```

```
$publicKey = file_get_contents('./public.pub');
$tokenEncoded = new TokenEncoded($tokenString);

try {
  $tokenEncoded->validate($key);
} catch (IntegrityViolationException $e) {
  // Token is not trusted
}
```

### Creating new token with expiration date (exp)

You may need to define expiration date for your token. To do so, you need to provide timestamp of expiration date into token's header under ```exp``` key.

```
$tokenDecoded = new TokenDecoded(['exp' => time() + 1000], $payload);
$tokenEncoded = $tokenDecoded->encode($key);
```

### Creating new token with not before date (nbf)

You may need to define date before which your token should not be valid. To do so, you need to provide timestamp of not before date into token's header under ```exp``` key.

```
$tokenDecoded = new TokenDecoded(['nbf' => time() + 1000], $payload);
$tokenEncoded = $tokenDecoded->encode($key);
```

### Creating new token with issued at date (iat)

This is simillar to ```nbf```, but here you need to provide timestamp of issued at date into token's header under ```iat``` key.

```
$tokenDecoded = new TokenDecoded(['iat' => time() + 1000], $payload);
$tokenEncoded = $tokenDecoded->encode($key);
```

### Solving clock differences issue between servers (exp, nbf, iat)

Because clock may vary across the servers, you can use so called ```leeway``` to solve this issue when validating token. It's some kind of margin which will be taken into account when validating token (```exp```, ```nbf```, ```iat```).

```
$leeway = 500;
$tokenEncoded = new TokenEncoded($tokenString);
$tokenEncoded->validate($key, $leeway);
```