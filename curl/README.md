This directory contains a Dockerfile that builds curl with the [OQS OpenSSL 1.1.1 fork](https://github.com/open-quantum-safe/openssl), which allows curl to negotiate quantum-safe keys and use quantum-safe authentication in TLS 1.3.

## Quick start

1) Be sure to have [docker installed](https://docs.docker.com/install).
2) Run `docker build -t oqs-curl .` to create a post quantum-enabled OpenSSL and Curl docker image
3) To verify all components perform quantum-safe operations, first start the container with `docker run -it oqs-curl` thus starting an OQS-enabled TLS test server.
4) On the command prompt in the docker container query that server using `curl https://localhost:4433`. If all works, the last command returns all TLS information documenting use of OQS-enabled TLS.


## More details

The Dockerfile 
- obtains all source code required for building the quantum-safe crypto (QSC) algorithms, the QSC-enabled version of OpenSSL (v.1.1.1), curl (v.7.66.0) 
- builds all libraries and applications
- creates OQS-enabled certificate files for a mini-root certificate authority (CA) 
- creates an OQS-enabled server certificate for running a `localhost` QSC-TLS server
- by default starts an openssl (s_server) based test server.

The signature algorithm for the certificates is set to `dilithium2` by default, but can be changed to any of the [supported OQS signature algorithms](https://github.com/open-quantum-safe/openssl#authentication) with the build argumemt to docker `--build-arg SIG_ALG=`*name-of-oqs-sig-algorithm*, e.g. as follows:
```
docker build -t oqs-curl --build-arg SIG_ALG=qteslapiii .
```

**Note for the interested**: The build process is two-stage with the final image only retaining all executables, libraries and include-files to utilize OQS-enabled curl and openssl.

Two further, runtime configuration option exist that can both be optionally set via docker environment variables:

1) Setting the key exchange mechanism (KEM): By setting 'KEM_ALG' 
to any of the [supported KEM algorithms built into OQS-OpenSSL](https://github.com/open-quantum-safe/openssl#key-exchange) one can run TLS using a KEM other than the default algorithm 'kyber512'. Example: `docker run -e KEM_ALG=newhope1024cca -it oqs-curl`. 

2) Setting the signature algorithm (SIG): By setting 'SIG_ALG' to any of the [supported OQS signature algorithms](https://github.com/open-quantum-safe/openssl#authentication) one can run TLS using a SIG other than the one set when building the image (see above). Example: `docker run -e SIG_ALG=picnicl1fs -it oqs-curl`.

## Easter egg

The docker image can also be used to execute performance tests against the different OQS algoritms: Simply start the image as shown above and then execute the standard OpenSSL handshake performance test like this `openssl s_time -connect :4433`. Be sure to first define the algorithms you are interested in, e.g., `docker run -e KEM_ALG=sikep751 -e SIG_ALG=picnicl1fs -it oqs-curl`.
