##Entropy (or lack of)
If your application appears to wait for no reason at all, it may be because there is not enough entropy available to provide randomness for /dev/random. I have experienced this while installing a applicaiton, where the application was waiting for so long that the oracle database was resetting the TCP socket (java.net.SocketException: Connection reset).

Best article I found on the interweb that describe the issue and fix is here.

https://www.certdepot.net/rhel7-get-started-random-number-generator/


