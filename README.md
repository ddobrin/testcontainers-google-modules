Examples using the [Testcontainers GCloud module](https://www.testcontainers.org/modules/gcloud/) to test applications locally.

All examples leverage the Google Cloud SDK and emulators provided by Google for several services.

The repository provides examples for:
1. Cloud Spanner - plain Java, no framework in use - directly leverage Testcontainers
2. Cloud Spanner - SpringBoot in use - leverage Testcontainers
3. Postgres (Cloud SQL) - SpringBoot and Spring Cloud GCP 2.x in use - latest released version, as maintained at this time by Google Cloud Teams

To run all the tests:
* From the repo root:  `mvn verify`
* From each individual sample: `mvn verify`

To run integration tests as part of a CI/CD build:
```shell
# Create a repository in Artifact Registry - let's call it: emulators
> gcloud artifacts repositories create emulators --repository-format=docker --location=us-central1 --description="Docker repository Testcontainer and emulator test images"

# Submit the CloudBuild pipeline and observe the integration tests being executed

# Spanner, plain Java
> gcloud builds submit --config=noframeworks/spanner-example/cloudbuild.yaml .

# Spanner, SpringBoot
> gcloud builds submit --config=springboot-gcp/spanner-example/cloudbuild.yaml .

# Postgres, SpringBoot
> gcloud builds submit --config=noframeworks/postgres-example/cloudbuild.yaml .
```

Each CloudBuild manifest shows the following build steps, each running individually in a container
```shell
steps:
  # Run Integration tests in Containers
  # fail-fast by running integration tests before packaging the app
  # Runs > mvn verify
  - name: maven:3-jdk-11
    entrypoint: mvn
    args: ["verify"]
    dir: 'noframeworks/spanner-example'

  # Testing completed --> Build the application
  # Do not run tests when building the app, tests have been executed
  - name: maven:3-jdk-11
    entrypoint: mvn
    args: ["package", "-Dmaven.test.skip=true"]
    dir: 'noframeworks/spanner-example'

  # Build the image
  - name: gcr.io/cloud-builders/docker
    args: ["build", "-f", "Dockerfile", "-t", "us-central1-docker.pkg.dev/$PROJECT_ID/emulators/spanner-example", "."]
    dir: 'noframeworks/spanner-example'

  # Push the container image to Artifact Registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'us-central1-docker.pkg.dev/$PROJECT_ID/emulators/spanner-example']

```