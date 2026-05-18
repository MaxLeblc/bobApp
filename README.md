# BobApp

Clone project:

> git clone XXXXX

## Front-end 

Go inside folder the front folder:

> cd front

Install dependencies:

> npm install

Launch Front-end:

> npm run start;

### Docker

Build the container:

> docker build -t bobapp-front .  

Start the container:

> docker run -p 8080:8080 --name bobapp-front -d bobapp-front

## Back-end

Go inside folder the back folder:

> cd back

Install dependencies:

> mvn clean install

Launch Back-end:

>  mvn spring-boot:run

Launch the tests:

> mvn clean install

### Docker

Build the container:

> docker build -t bobapp-back .  

Start the container:

> docker run -p 8080:8080 --name bobapp-back -d bobapp-back

## CI/CD Pipeline (GitHub Actions & SonarQube)

This project implements a complete Continuous Integration (CI) and Continuous Deployment (CD) pipeline using GitHub Actions. The workflow is configured to run automatically with every `push` or `pull_request` to the `main` branch.

### Workflow Architecture

The pipeline is located in the `.github/workflows/ci.yml` file and consists of three key sequential steps:

1. **Validation & Testing (CI):**
   * **Back-end:** Compilation and execution of Java unit/integration tests via Maven (`mvn clean verify`). Automatic generation of the code coverage report with Jacoco.
   * **Front-end:** Installation of dependencies via NPM and execution of Angular unit tests in headless mode (`npm test -- --browsers=ChromeHeadless --watch=false`). Generation of the coverage report in LCOV format.
   * *Persistence:* Coverage reports are extracted and shared as workflow artifacts.

2. **Quality Audit (SonarQube Cloud):**
   * This job (`sonar-analysis`) runs only if the testing stage passes.
   * It downloads the coverage artifacts, recompiles the Java code (`mvn compile`) to provide the required `.class` binary files, and then runs the SonarQube scanner.
   * The analysis verifies compliance with the Quality Gate (Overall Coverage, bug detection, vulnerabilities, and *Code Smells*).

3. **Packaging & Deployment (CD):**
   * This job (`deploy-docker`) runs only if the SonarQube Quality Gate is passed.
   * It relies on multi-stage build architectures to compile and isolate deliverables within lightweight, secure Docker images based on **Alpine** distributions (`eclipse-temurin:11-jdk-alpine` for the back-end and `nginx:alpine` for the front-end).
   * The built images are automatically pushed to Docker Hub.

### Required Environment Variables and Secrets

To function, the pipeline requires the following secrets to be configured in your GitHub repository settings (`Settings > Secrets and variables > Actions`):

* `SONAR_TOKEN`: The authentication token provided by SonarQube Cloud for project analysis.
* `DOCKER_USERNAME`: Your unique Docker Hub username.
* `DOCKER_PASSWORD`: A Personal Access Token (PAT) generated on Docker Hub with **Read & Write** permissions.

### Manual Triggering and Monitoring

* To monitor pipeline execution in real time, go to the **Actions** tab of your GitHub repository.
* Once the pipeline is approved (green), the updated production Docker images become immediately available on your public Docker Hub profile and can be retrieved using standard infrastructure commands:
  ```bash
  docker pull <your-username>/bobapp-back:latest
  docker pull <your-username>/bobapp-front:latest
