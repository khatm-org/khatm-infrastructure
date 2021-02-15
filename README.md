# Ihsan Infrastructure as Code

- [The Ihsan Project](https://ihsanproject.com/)
- [The Infrastructure Architecture](https://github.com/ihsan-project/ihsan-infra-pulumi/wiki/Architecture)

## Setup Environment
1. Create an account with [Pulumi](https://app.pulumi.com/signin?reason=401).
    - There's an option to host your own. Look into this for future projects.
1. Update the `Pulumi.yaml` with the project name.
1. Install the pulumi CLI and login to account
    - `brew install pulumi`
    - `pulumi login`
    - Enter token from the Pulumi dashboard: https://app.pulumi.com/account/tokens
1. Add the following environment variables to your `~/.bash_profile` or `~/.zshrc`. For some reason it doesn't work in the `.env`
    - CLOUDFLARE_EMAIL. This is the user's account, not the owner account
    - CLOUDFLARE_API_KEY. This is the Global API Key from https://dash.cloudflare.com/profile/api-tokens
1. Run `cp .env-keep .env` and fill in each key following the provided hints
1. In AWS Secrets Manager, setup the following key/values:
    - {APP_NAME}/{Pulumi Stack}/google_sso_client_id. For example: `ihsan/staging/google_sso_client_id`
    - {APP_NAME}/{Pulumi Stack}/api_key
    - {APP_NAME}/{Pulumi Stack}/db/name.
    - {APP_NAME}/{Pulumi Stack}/db/user.
    - {APP_NAME}/{Pulumi Stack}/db/password.
    - Instructions:
        1. Navigate to AWS Secrets Manager
        1. Choose `Other types of secrets`
        1. Choose `Plaintext`. Insert the value here only as plain text, don't use JSON.
        1. Next screen, here you enter the key for the `Secret Name`. Eg. `ihsan/staging/google_sso_client_id`.
1. Create an AWS ECR repository
    1. Go to AWS ECR
    1. Click Create Repository, give it any name
1. Install [Docker Engine](https://docs.docker.com/get-docker/)
    - Docker Desktop is fine when running on local machine

## Run Commands

Currently it's quite a task to seperate foundational services from application services (because we don't want to bring down our database and lose production data when we want to restart the servers) through Pulumi ([Issue 27](https://github.com/ihsan-project/ihsan-infra-pulumi/issues/27)), we're using node scripts and environment variables to control what part of the code runs.

The following are to start and stop the foundation services (VPC, RDS, etc)
- `npm run start:foundation`
- `npm run stop:foundation`

The following are to start and stop the application services (ECS, ALB, DNS entries, etc)
- `npm run start:application`
- `npm run stop:application`

Note: Stopping foundation will also stop application services.

## Debug

Common failures seen:
- `pulumi up` fails on the CloudFormation stack is in `ROLLBACK_COMPLETE` or `DELETE_COMPLETE` state `and can not be updated`.
    - Solution: Go into AWS Console and delete the offensive CloudFormation instance.
    - Run `pulumi refresh` and let Pulumi recognize state and update the Pulumi stack.
    - Try again
- `pulumi destroy` fails to delete a service with dependencies.
    - Run `pulumi stack export | pulumi stack import`
    - Run `pulumi refresh`
    - Try again
