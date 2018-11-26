# circleci-orbs &middot; [![CircleCI](https://circleci.com/gh/LedgerHQ/circleci-orbs.svg?style=shield)](https://circleci.com/gh/LedgerHQ/circleci-orbs)
CircleCI orbs maintained by LedgerHQ

## Docker

[Orb documentation](https://circleci.com/orbs/registry/orb/ledger/docker)

This orb implements the following actions :
- it builds a Docker image (the Dockerfile is expected to be stored at the root of the repository).
- it leverages the [goss](#note-about-goss) tool (more precisely it uses the dgoss wrapper) to check if the image is properly working. 
To be able to test under adequate conditions, it may use Docker Compose to launch a complete environment powering all the needed service dependencies (database, third-party component).
- it publishes the image on the Docker Hub registry. It sets the following Docker tags : the commit SHA1 and either the branch name (for a commit-triggered CI run) or the Git tag (for a tag-triggered CI run).

### Note about goss
[Goss](https://github.com/aelsabbahy/goss) is a self-sufficient tool that allows to easily and quickly execute a sequence of checks like 
testing if a process is running, testing if a port is listening, testing the return status of a command, querying a HTTP server and
[much more](https://github.com/aelsabbahy/goss/blob/master/docs/manual.md#available-tests).

An additionnal wrapper named [dgoss](https://github.com/aelsabbahy/goss/tree/master/extras/dgoss) and shipped with the project brings a smooth integration with Docker.
As goss can output its result in JUnit format, it integrates pretty well with the [CircleCI interface](https://circleci.com/docs/2.0/collect-test-data/).
