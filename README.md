# circleci-orbs &middot; [![CircleCI](https://circleci.com/gh/LedgerHQ/circleci-orbs.svg?style=shield)](https://circleci.com/gh/LedgerHQ/circleci-orbs)
CircleCI orbs maintained by LedgerHQ

## Chef

[Orb documentation](https://circleci.com/orbs/registry/orb/ledger/chef)

This orb implements the following actions on a Chef cookbook :
- it validates that all files pertaining to a cookbook are correct :
  - it checks that the cookbook version is valid and has been increased. See the [note about the cookbook version](#note-about-the-cookbook-version).
  - it checks that all JSON files are valid.
  - it checks the Ruby syntax with Rubocop or Cookstyle.
  - it checks the Chef syntax with Foodcritic.
- it tests the cookbook by running it with Kitchen.
- it uploads the cookbook on a Chef Server with Berkshelf. To use that job, [a Chef client must first be created and allowed to upload cookbooks](#creation-of-a-Chef-client-allowed-to-upload-cookbooks).
  The job takes care to exclude [fixture cookbooks defined inside a group named "integration"](https://kitchen.ci/docs/reference/fixtures/).

### Note about the cookbook version

The cookbook version is expected to be read from a file named `VERSION` and located at the root of the repository.

Here is an example of cookbook configuration :
- metadata.rb :
    ```
    version IO.read(File.join(File.dirname(__FILE__), 'VERSION'))
    ```
- VERSION :
    ```
    0.0.1
    ```

### Creation of a Chef client allowed to upload cookbooks

A Chef client can easily be created with Knife :

```sh
knife client create -f circleci.pem -d circleci
knife acl add client circleci containers cookbooks create
knife acl add client circleci containers sandboxes create
knife acl bulk add client circleci cookbooks ".*" update
```

The RSA key needs to be provided to CircleCI so that it can authenticate to the Chef Server.
The orb retrieves the private key in base64 format (needed for storing multi-line data) by looking for a CircleCI environment variable named `CHEF_KEY` :

```sh
base64 -w 0 circleci.pem
```

In the context of the Chef Server's API a container is just the API endpoint used when creating a new object of a particular object type.
Two containers are used when creating (uploading) new cookbooks : the cookbooks and sandboxes containers.

For reference, here is an explanation about the sandbox container use :
> A Sandbox is a temporary list of files that you intend to upload. The actual
> files you upload are stored in an S3-alike service (bookshelf) or real S3
> (Hosted Chef does this, you can configure it on your own server as well if
> you wish). Therefore Chef Server needs a mechanism to know what files you’ve
> promised to upload while it waits for you to upload them to a separate
> service. That’s what the sandbox does.

[source](https://discourse.chef.io/t/upload-create-cookbooks-using-client/7222/14)

## Docker

[Orb documentation](https://circleci.com/orbs/registry/orb/ledger/docker)

This orb implements the following actions :
- it builds a Docker image (the Dockerfile is expected to be stored at the root of the repository).
- it leverages the [goss](#note-about-goss) tool (more precisely it uses the dgoss wrapper) to check if the image is properly working. 
To be able to test under adequate conditions, it may use Docker Compose to launch a complete environment powering all the needed service dependencies (database, third-party component).
- it publishes the image on the Docker Hub registry. It sets the following Docker tags : the commit SHA1 and either the branch name (for a commit-triggered CI run) or the Git tag (for a tag-triggered CI run).

To be able to publish to the Docker Hub registry, you have to define the following environment variables on the CircleCI project settings :
- DOCKER_USERNAME
- DOCKER_PASSWORD
- DOCKER_ORGANIZATION if the repository is managed by a Github organization

**BEWARE OF PUBLIC REPOSITORIES** : if you allow CircleCI to run builds from forked pull request, you must take care of not sharing these environment variables to forked pull request as this will allow anyone to retrieve your Docker Hub credentials. Check that the settings *Pass secrets to builds from forked pull requests* is disabled.

### Note about goss

[Goss](https://github.com/aelsabbahy/goss) is a self-sufficient tool that allows to easily and quickly execute a sequence of checks like 
testing if a process is running, testing if a port is listening, testing the return status of a command, querying a HTTP server and
[much more](https://github.com/aelsabbahy/goss/blob/master/docs/manual.md#available-tests).

An additionnal wrapper named [dgoss](https://github.com/aelsabbahy/goss/tree/master/extras/dgoss) and shipped with the project brings a smooth integration with Docker.
As goss can output its result in JUnit format, it integrates pretty well with the [CircleCI interface](https://circleci.com/docs/2.0/collect-test-data/).
