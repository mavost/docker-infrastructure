# A simple base of docker infrastructure

Revision: August 2022

**keywords**: software-architecture

*contacts*: Markus von Steht

## Description

The stack contains the following modules:

- [Portainer](https://www.portainer.io/) to monitor and administer what is happening with docker
- [Traefik](https://traefik.io/) to monitor and administer the networking namespace

## Running the Docker compose pipeline

1. Copy `.env.dist` to `.env` (no adjustments required / for future use with credentials)
2. Run `make run-compose` and let the container for the *simple* stack come online.
3. Use <kbd>CTRL</kbd>+<kbd>C</kbd> to shut down the stack
4. Invoke `make clean` and `make clean stack=extended`, respectively to remove the stack

### Manual access to container

Using compose:  
`docker-compose exec <container-name> env SAMPLEPAR="testing" bash`

Using docker:  
`docker exec -it <container-name> bash`

### Installing self-signed certs and enabling TLS in Traefik

1. `sudo apt install libnss3-tools`
2. Download executable

    ```bash
    curl -JLO "https://dl.filippo.io/mkcert/latest?for=linux/amd64"
    chmod +x mkcert-v*-linux-amd64
    sudo cp mkcert-v*-linux-amd64 /usr/local/bin/mkcert
    rm mkcert-v*-linux-amd64
    ```

3. Running `mkcert -install`, results in *"Created a new local CA"*.
The file `rootCA.pem` will usually be generated in the folder `$HOME/.local/share/mkcert` which can be confirmed using `mkcert -CAROOT`.
4. (Optional) Copy `rootCA.pem` to relevant area of operations and register with trust store or install in browser.
5. Build certificates for specific endpoints with or without wildcards:

    ```bash
    mkcert -cert-file local-cert.pem -key-file local-key.pem example.com "*.example.com" example.test localhost 127.0.0.1
    ::1
    ```

    ```bash
    # output
    Created a new certificate valid for the following names
    - "example.com"
    - "*.example.com"
    - "example.test"
    - "localhost"
    - "127.0.0.1"
    - "::1"
    Reminder: X.509 wildcards only go one level deep, so this won\'t match a.b.tufhades-local.net ℹ️
    The certificate is at "./local-cert.pem" and the key at "./local-key.pem"
    ```

6. (Optional) Copy `local-cert.pem` and `local-key.pem` to reverse proxy config staging location and bind them to the container.
7. Configure Traefik `dynamic-conf.yaml` to match the domains used in the certificate and ensure that the certs file are properly referenced from the location to which they will be copied in the container.
8. Ensure that TLS/SSL ports are open for receiving traffic and that endpoint labels in container configurations have TLS enabled and proper routing configured, e.g.:

    ```bash
    # docker-compose.yaml
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.tls=true"
      - "traefik.http.routers.traefik.rule=Host(`traefik.${DOMAIN_NAME}`)"
      - "traefik.http.services.traefik.loadbalancer.server.port=8080"
    ```

References:

- [Mkcert](https://github.com/FiloSottile/mkcert)
- [Configuration of TLS](https://knplabs.com/en/blog/how-to-handle-https-with-docker-compose-and-mkcert-for-local-development/)
