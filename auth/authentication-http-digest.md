 HTTP Digest Configuration
=========================

HTTP Digest authentication provides similar setup requirements to HTTP Basic, and adds the benefit
that passwords are not sent over the network in plain text. The tool used to create a proper digest
file also comes with the [Apache](http://httpd.apache.org/) installation: `htdigest`. If this tool
is not present on your system, there are a number of web based tools that will also produce a valid
`htpasswd` file; [google for "htdigest generator"](https://www.google.com/search?q=%22htdigest+generator%22) for examples.

Like HTTP Basic authentication, a digest file will need to exist before configuration of this
authentication scheme takes place:

```console
$ htdigest -c data/users.htdigest "Secure API" ralph
Adding password for ralph in realm Secure API
New password:
Re-type new password:
$
```

Once the file has been created, its path can be used to configure the required `htdigest` file input
of the HTTP Digest authentication configuration screen, shown here:

![Configuring HTTP Digest settings](/asset/apigility-documentation/img/auth-authentication-http-digest-ui-settings.jpg)

The configuration data will be stored in `config/autoload/local.php` under the key
`['zf-mvc-auth']['authentication']['adapters']['digest']` where `digest` is the name of the adapter chosen
in the previous screenshot. You can have many authentication adapters in your configuration file,
and choose the one to be used for your API. Below is an example of config with two authentication adapters
(HTTP Basic and Digest).

```php
// config/autoload/local.php
return [
    'zf-mvc-auth' => [
        'authentication' => [
            'adapters' => [
                'basic' => [
                    'adapter' => 'ZF\\MvcAuth\\Authentication\\HttpAdapter',
                    'options' => [
                        'accept_schemes' => [
                            0 => 'basic',
                        ],
                        'realm' => 'api',
                        'htpasswd' => 'data/users.htpasswd',
                    ],
                ],
                'digest' => [
                    'adapter' => 'ZF\\MvcAuth\\Authentication\\HttpAdapter',
                    'options' => [
                        'accept_schemes' => [
                            0 => 'digest',
                        ],
                        'realm' => 'Secure API',
                        'digest_domains' => '',
                        'nonce_timeout' => '3600',
                        'htdigest' => 'data/users.htdigest',
                    ],
                ],
            ],
        ],
    ],
];
```

At this point, HTTP Digest authentication has been setup and is ready to use.

```HTTP
GET /foo HTTP/1.1
Accept: application/json
Authorization: Digest username="ralph", realm="Secure API", nonce="2f3fdb4e7670ae34f0b5c092d720961c", uri="/foo", response="eaaf8d5ac41022635277a4196b747ba1", opaque="e66a41ca5bf6992a5479102cc787bc9", algorithm="MD5", qop=auth, nc=00000001, cnonce="c07b87e1b0cc5115"


```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json

{
    "foo": "bar"
}
```

And a failed attempt at authentication:

```HTTP
GET /foo HTTP/1.1
Accept: application/json
Authorization: Digest clearly-invalid-token


```

```HTTP
HTTP/1.1 401 Unauthorized
Content-Type: application/problem+json

{
    "type": "http://www.w3c.org/Protocols/rfc2616/rfc2616-sec10.html",
    "title": "Unauthorized",
    "status": 401,
    "detail": "Unauthorized"
}
```


Important Notes
---------------

- Your client should be capable of properly encoding the HTTP Digest `Authorization` header, and
  able to fulfill the digest handshake.
- In production, ensure an `htdigest` file can be utilized in the same relative location as in
  development, even if the `htdigest` was not checked into your VCS.
- No `Authorization` header in the request implies that the "guest" `ZF\MvcAuth\Identity\GuestIdentity`
  identity will be used.
