# A simple base of docker infrastructure

Revision: August 2022

**keywords**: software-architecture

*contacts*: Markus von Steht

## Description

The stack contains the following modules:

- [Portainer](https://www.portainer.io/) to monitor and administer what is happening with docker
- [Treafik](https://traefik.io/) to monitor and administer the networking namespace

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
