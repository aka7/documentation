
## view details of a pem file.
You can view PEM formatted certificate under Linux like so
````
openssl x509 -in cacert.pem -text
````

## view certfificate details of a site
````
openssl s_client -connect google.com:443  -showcerts
````
## view expiry, subject, issuer of a site, example below is for ldaps
````
echo | openssl s_client -connect 10.10.10.10:636 |  openssl x509 -noout -dates -subject -issuer
````
