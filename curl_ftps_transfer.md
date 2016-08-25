# uploading a file using curl using ftps on port 8022
confirued vsftpd for tls conenction, 
following command is useful to test the tls connection and upload a file using curl
````
curl --cacert /etc/ssl/testca.pem --ftp-ssl --tlsv1 -T  fileToTransfer ftp://vsftp:PASSWORD@hostexmaple.com:8022/
````
