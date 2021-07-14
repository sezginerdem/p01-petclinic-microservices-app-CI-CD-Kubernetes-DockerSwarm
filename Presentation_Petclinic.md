# Presentation Short Version

## Description
***
This project aims to create full CI/CD Pipeline for microservice based applications using [Spring Petclinic Microservices Application](https://github.com/spring-petclinic/spring-petclinic-microservices). Jenkins Server deployed on Elastic Compute Cloud (EC2) Instance is used as CI/CD Server to build pipelines.

## Abstract
***
This project is a Java-based petclinic-microservis web application developed by Spring company. The frontend part of the application was written in React and the backend part was written in Java.

The application has 3 main menus: **Home**, **owners** and **veterenians**.

The project consists of 8 microservices and there is also a monitoring service with Prometheus and Grafana. These services are:
  1. Admin-server
  2. Api-gateway(UI  api-gateway)
  3. Custom­er-server
  4. Cofig-server
  5. Discovery-server
  6. Hystrix Dashboard
  7. Vet Server
  8. Visit Service

It is a cloud native application using Fly database.

### Development Diagram
***

![Development Diagram](./project-501-dev-diagram.png)

## Expected Outcome
***
Automate the build, unit test, deploy, functional test and deploy to production stages of the application by creating pipelines for unit test, functional tests, manual QA, staging and production environments. 

Bunun icin 5 pipeline kurdum:  
**1. Pipeline-ci-job:** dev, feature ve bugfix branslari icin webhookla trigger edecek sekilde maven ile build edilecek ve jacoco, unit testlerini yapmak amaciyla pipeline kuruldu.
    
**2. Pipeline-nightly:** dev bransi icin functional testleri yapmak uzere docker-swarm uzerinde deploy edilmis environment da nightly-cronjob ile build, unit test, deploy ve functional testler yapilacak. Stable dir ancak tum testlerden gecmemis versioyondur.
    
**3. Pipeline-weekly:** Release bransindaki kodu her hafta pazar gunu build, unit test, deploy edildi. Manuel testerlar manuel testlerini bu enviroment da gerceklestirecek. 
    
**4. Pipeline-staging:** Release bransinda kod her hafta pazar gunu build, unit test, functional test, manuel testerlar manuel testleri yapacak v staging env a gidecek. Bundan sonrasi kod ile ne yapilacagi ile ilgili. Alfa, beta surecine gidilebilir, User acceptance testlerini yapmak icin musteriye sunulabilir. 
    
**5. Pipeline-prod:** Master brancinda webhook ile trigger ederek yapacagimiz her commitde kodumuza build, unit test, deploy ve functional test asamalarini uygulayarak musteriye sunulacak production ortamina deploy edecek. 

CI-job, nighty ve weekly pipeline lar docker swarm ile orchestrate edilmis iken staging ve prod pipeline lari ise kubernetes ile orchestrate edildi.

### Pipelines Configurations
***
![Pipelines to be configured](./project-501-pipelines.png)

## Projede Kullanilan Toollar
***
* **Programming Language:** Bash Script, Groovy
* **Infrastucture as Code:** Ansible, CloudFormation
* **Containerisation:** Docker
* **Container Orchestiration:** Kubernetes, Docker Swarm
* **CI/CD Pipeline:** Jenkins
* **Cloud:** AWS (EC2, VPC, EBS, IAM, ECR, AMI)
* **Monitorong:** Prometheus, Grafana
* **Version Control:** Git, GitHub
* **Build**: Maven
* **Others:** Docker-Compose, Helm, Rancher