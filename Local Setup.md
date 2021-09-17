# GitOps Lab

## Setup

Run `docker-compose up`.

Wait for LocalStack services to be ready.
```
localstack_gitops | Waiting for all LocalStack services to be ready
localstack_gitops | Ready.
```

Run the following command to list S3 buckets
```
docker run \
  --network gitops_localstack \
  --env AWS_ACCESS_KEY_ID=mock \
  --env AWS_SECRET_ACCESS_KEY=mock \
  --rm -it amazon/aws-cli \
  --endpoint-url=http://localstack:4566 s3api list-buckets
```

## Run

Initialize the Terraform configuration
```
docker run \
  --env AWS_ACCESS_KEY_ID=mock \
  --env AWS_SECRET_ACCESS_KEY=mock \
  --volume `pwd`:/infra \
  --workdir /infra \
  --rm -it hashicorp/terraform init
```

Validate the Terraform configuration
```
docker run \
  --env AWS_ACCESS_KEY_ID=mock \
  --env AWS_SECRET_ACCESS_KEY=mock \
  --volume `pwd`:/infra \
  --workdir /infra \
  --rm -it hashicorp/terraform validate
```

Create an execution plan
```
docker run \
  --env AWS_ACCESS_KEY_ID=mock \
  --env AWS_SECRET_ACCESS_KEY=mock \
  --volume `pwd`:/infra \
  --workdir /infra \
  --rm -it hashicorp/terraform plan
```

Create the infrastructure
```
docker run \
  --network gitops_localstack \
  --env AWS_ACCESS_KEY_ID=mock \
  --env AWS_SECRET_ACCESS_KEY=mock \
  --volume `pwd`:/infra \
  --workdir /infra \
  --rm -it hashicorp/terraform apply
```

Build image to run GitHub Actions locally
```
docker build \
  -t nektos/act \
  -f ./act/Dockerfile \
  ./act
```

Run GitHub Actions locally
```
docker run \
  --network gitops_localstack \
  --volume `pwd`:/actions \
  --volume /var/run/docker.sock:/var/run/docker.sock \
  --workdir /actions \
  --rm -it nektos/act \
  --platform ubuntu-latest=ghcr.io/catthehacker/ubuntu:act-latest
```

## Cleanup

Stop LocalStack.

Clean up the Docker Compose stack with `docker-compose down`.