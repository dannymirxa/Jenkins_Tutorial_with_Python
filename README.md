# FastAPI-App-Example

## 1. Objectives

### 1.1 To deploy a FASTApi app

Repository's URL: `https://github.com/dannymirxa/FastAPI-App-Example/`

### 1.2 To learn how to use Jenkins for automation

## 2. Custom Docker Images and Containers

### 2.1 Write the dockerfile

This dockerfile create a new PosgreSQL image and run `init.sql` during initialization.

```dockerfile
FROM postgres:latest

ENV POSTGRES_USER=myuser
ENV POSTGRES_PASSWORD=mypassword
ENV POSTGRES_DB=Employees

COPY init.sql /docker-entrypoint-initdb.d/

EXPOSE 5432
```

This dockerfile run the FastAPI app.

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY *.py .

EXPOSE 8000

CMD ["python", "main.py"]

```

### 2.2 Build the docker images

```bash
docker build -f posgres-dockerfile -t employee-posgres . 
```

```bash
docker build -f fastapi-dockerfile -t fastapi-employee . 
```

### 2.3 Write the compose Yaml named posgres-compose.yaml

```yaml
services:
  postgres:
    image: employee-posgres
    environment:
      - POSTGRES_USER=myuser
      - POSTGRES_PASSWORD=mypassword
      - POSTGRES_DB=Employees
    ports:
      - "5432:5432"
    volumes:
      - db-data:/var/lib/postgresql/data
    container_name: employee_db
volumes:
  db-data:
```

### 2.4 Write the compose Yaml named fastapi-compose.yaml

```yaml
services:
  fastapi_app:
    image: fastapi-employee
    ports:
      - "8000:8000"
    volumes:
      - .:/app
    environment:
      - ENV=production
    container_name: employee_app
```

### 2.4 Run the docker compose command

```bash
docker compose -f posgres-compose.yaml up
```

```bash
docker compose -f fastapi-compose.yaml up
```

## 3. Jenkins Steps

### 3.1 Build Simple Testing Pipeline

#### 3.1.1 Build a simple pipeline by clcking 'New Item'

<!-- ![alt text](./Images/Test%20Pipeline.png) -->

<img src="./Images/Test Pipeline Create.png" alt="Alt Text" width="1000">

#### 3.1.2 Write a Pipeline Script

For now just print 'Testing Pipeline with Jenkins'. Done forget to click 'Save'.

<img src="./Images/Test Pipeline Script.png" alt="Alt Text" width="1000">

#### 3.1.3 Click 'Build Now'

<img src="./Images/Test Pipeline Build.png" alt="Alt Text" width="500">

#### 3.1.4 Check the Build Result

<img src="./Images/Test Pipeline Build Result.png" alt="Alt Text" width="500">

#### 3.1.5 Check the Build Stage

<img src="./Images/Test Pipeline Stages.png" alt="Alt Text" width="500">

### 3.2 Build Simple Multi-Stage Testing Pipeline

#### 3.2.1 Just continue from Step 3.1.2, update the Pipline script

<img src="./Images/Test Pipeline Multistage Script.png" alt="Alt Text" width="500">

#### 3.2.2 Check the stages

<img src="./Images/Test Pipeline Multistage Stages.png" alt="Alt Text" width="500">


## 4. Jenkins Pipeline

### 4.1 Create Github Token

This steps is to let Jenkins have access to our Github repository. Navigate to settings and create classic token.

<img src="./Images/Create Github Token.png" alt="Alt Text" width="500">

### 4.2 Create Jenkins Github credential

After creating the token, navigate to credential section in Jenkins configuration as in the image:

<img src="./Images/Create New Github Credentials in Jenkins.png" alt="Alt Text" width="500">

Use email as Username and put the token generated to the Password.

### 4.3 Pipeline syntax

If you want to starts with a simple pipline syntax, navigate trough here:

#### 4.3.1 Click the button `Pipeline Syntax` when creating a pipeline

<img src="./Images/Example Pipeline Syntax: Checkout.png" alt="Alt Text" width="500">

#### 4.3.1 Select the sample steps, fill in the parameters and generate the syntax

<img src="./Images/Use Pipeline Syntax.png" alt="Alt Text" width="500">

### 4.4 Jenkinsfile explanation

```Jenkinsfile
pipeline {
    agent any
    stages {
        ### Checkout the repository
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: 'main']], extensions: [], userRemoteConfigs: [[credentialsId: 'fastapi-app', url: 'https://github.com/dannymirxa/FastAPI-App-Example.git']])
            }
        }
        ### Clone the repository
        stage('Build') {
            steps {
                git branch: 'main', credentialsId: 'fastapi-app', url: 'https://github.com/dannymirxa/FastAPI-App-Example.git'
            }
        }
        ### Steps below is identical as
        ### docker build -t fastapi-test:latest -f pytest-dockerfile .
        ### docker run fastapi-test
        stage('Test') {
            steps {
                script {
                    docker.build('fastapi-test:latest', '-f pytest-dockerfile .').inside {
                        sh 'pytest test_main.py'
                    }
                }
            }
        }
        
    }
}

```
