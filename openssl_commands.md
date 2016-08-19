
## view details of a pem file.
You can view PEM formatted certificate under Linux like so
````
openssl x509 -in cacert.pem -text
````

## view detail of a certfificate of a site
````
openssl s_client -connect google.com:443  -showcerts
````
