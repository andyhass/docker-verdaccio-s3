
# docker-verdaccio-s3

Private NPM container that can backup to S3

The container will sync content down from S3 on start, and perform a backup sync every minute.  

By default, Verdaccio uses the container filesystem for storage - storage of user profiles/accounts and published packages, specifically.  If the container is terminated, this data is destroyed.  Using S3 to store/backup these files, we can minimize the risk of user data, user accounts, and published packages from being destroyed.  

## Requirements

 - Docker (running (on a Mac) locally, I'm a fan of [
Docker for Mac](https://www.docker.com/docker-mac)).
- NPM 6.0.1 or later

## Usage

 1. Build a new image.  ```cd``` into the root of the repository and run: ```docker build --tag verdaccio .```
 2. Run the image:
```
docker run --rm --name verdaccio -e S3_BUCKET=[your-s3-bucket] -e AWS_ACCESS_KEY_ID=[your-aws-secret-key-id] -e AWS_SECRET_ACCESS_KEY=[your-aws-secret-access-key] -p 80:4873 verdaccio
```

At this point, Verdaccio should be running from http://localhost/

## User Management

Users are provisioned manually by modifying an htpasswd file located at ```/verdaccio/conf/htpasswd``` in the Docker container.

 1. SSH into a running container: ``
docker exec -it verdaccio sh
``
 2. If an htpasswd file doesn't exist in ```/verdaccio/conf/```, create one: ```touch /verdaccio/conf/htpasswd```
 3. Add a definition in ```/verdaccio/conf/htpasswd``` for a new user.  Valid user/encrypted password definitions for htpasswd files can be generated [here.
](http://www.htaccesstools.com/htpasswd-generator/)
 4. To deprovision a user, simply remove their username/password from htpasswd.

## Configuring the NPM CLI

To authenticate with Verdaccio, run: ```npm login --registry http://localhost``` and enter the user credentials (valid credentials defined via the step above).

At this point, you should be able to successfully publish packages to Verdaccio and use it as your NPM registry.

**NOTE:**  As mentioned in https://github.com/verdaccio/verdaccio/issues/509, changes introduced in updates to the NPM CLI can cause certain NPM commands to not work with Verdaccio.  In the issue above, ```npm login``` was behaving in a way that was incompatible with Verdaccio using versions `5.4.x` and `5.5.x` of the NPM CLI.  Earlier and later versions of the NPM CLI have worked in a way that allows ```npm login``` to function in a way that's compatible with Verdaccio (hence the requirement for NPM 6.0.1 or later).  Still, it's slightly concerning that changes to the NPM CLI can be incompatible with Verdaccio.
