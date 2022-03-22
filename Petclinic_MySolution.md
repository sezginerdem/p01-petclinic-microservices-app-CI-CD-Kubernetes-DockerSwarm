# Petclinic Microservices My Solution


* Work/Git Flow tanimlandi, projede nasil yapilacagi konusunda aciklamalar yapildi.
* Back-endi java ile front-endi ise react ve html ile yazildi.
* Proje Spring firmasi tarafindan gelistirilmis Java tabanli bir pet klinik web uygulamasi. Uygulama bir veritabani kullanmiyor. Kapatildiktan sonra veriler kayboluyor. Program test amacli yapildigi icin kisitli ozelliklere sahip. Home, owners ve veterenians olmak uzere 3 ana menusu var. 
* Uygulamada 8 microservis bulunmakta(ayrica monitoring olarak prometheus ve grafana servisi bulunmakta)Bunlar:
  1. Admin-server
  2. Api-gateway(UI  api-gateway)
  3. Customer-server
  4. Cofig-server(applicationlarin configurasyonlarinin bulundugu servis, mesela portlar degisebilir buportconfigurasyonlarini config serverdan aliyor, bu servis her mikroservisin baslagic asamasindailkparametrelerini vermekten sorumlu, kendilerini baslatirken bu configurasyonlari almadan baslatamiyorlar)
  5. Discovery-server(servislerin health checkini yapiyor, her servis discovery servise registrysini yapiyorbir cesit dns servis gibi healthy olup olmadigini kontrol ediyor, cloud angostik denilen iste busisteminherhangi bir cloud servisine bagimli olmadan calisabilmesidir )
  6. Hystrix Dashboard
  7. Vet Server
  8. Visit Service
* Kaynak kodlar Git Repoda saklanmaktadir. Uygulamanin kendi Spring Cloud Config Serveri vardir. Cloudagnostik bir uygulamadir yani herhangi bir cloudda calisabilir. Cunku kendisine ait bir configserveridiscovery servisi ve load serveri var. Ilave bir configurasyon ihtiyaci yok. Bunlar opensourceapplicationdolayisiyla her cloud ortaminda calisabiliyor.
* Cloud Native/Agnostic Application=Herhangi bir cloud ortamina bagimliligi yok, hepsinde calisabilir. Kendi config serveri var, kendi discovery serveri var, kendi monitoring serveri var.
* Zipkin login server kullniliyor. Uygulamanin tum log kayitlari bunun uzerinde tutuluyor. Herhangi bir microservisde proplem varsa hepsini buradan gorebiliyoruz.
* Spring frameworkunun bir admin serveri var.
* Weri tabani yok. Uygulama kendine ait on the fly veri tabani kullaniliyor. Program kapaninca tum kaydedilen veriler gidiyor yalnizca sample veriler kaliyor.
* Monitoring kismi prometheus ve grafana ile yapildi.
* Bu uygulama spring framework unda build edilmistir. Mesela flask uygulamasi python developerlar tarafindan gelistirilmistir. Spring framework java icin kullanilan entriprice bir web frameworkudur.Robast bir uygulamadir. Xml formatta yazilmis. 
* Pom.xml dosyasi icinde maven kodlarinin oldugu dosyadir ve bizim projemizin merkezini olusturmaktadir. Projenin configurasyonuna dair pek cok bilgileri kapsiyor. Version, isim, javanin versiyonu library dependencies. Her bir micro servisin pom.xml I oldugu gibi projenin de bir pom.xml dosyasi var. Boylece herbir servis kendi basina calisabiliyor.
* **Discovery-server:** Ilk once calistiginda config-serverdan configurasyonlari aliyor ve calismaya basliyor. DNSservisi gibi load balancer yaparak microservislere yonlendiriyor. Servisler config serviceden configurasyonlarini aldiktan sonra euroka service e geliyorler ve registery yapiyorlar ben bu servisim diye. Ben bu servisim ben calisiyorum diye ve euroka bunlari listeden bakiyor registry yapiyor. 
* Vets-service application: Bu gercek bir microservis. Kendi pom.file i var. Bu servisin sadece get metodu var. 
* **admin-server:** Sistem properties, system details, metrics,configuration properties gibi tumverilerimicroservislerden toplar ve ui e iletir orada.
* Build test ve unit testlerimi(UI Test) jenkins server da yaptim ama local de de yapabilirdim.
* Unit test ve integration testler yapildiktan sonra qa test icin environment build ettim. Iki tane farkli qa test var manuel ve otomated test(selenium jobs) her aksam bir infrastructure yaptim ve bu qa testleri environment uzerinde test ettim eger her sey yolunda ise kapatiyorum environmenti. Bunlar icin dockerswarm kullandim.
* Manuel testler icin hazirladigim environment ise bir hafta acik kaldi. Weekly pipelinedi bu. 
* Bu Projede amacim pipeline olusturmak. Projenin kodunu once kendi github hesabima yukledim. Branchlar olusturdum Jenkinsle o kodlari github hesabimdan cektim ve build ettim. JUnit ile yazilmis 3 tane unit test yazilmis onlarla unit testlerimi gerceklestirdim. Unit test asamasi basari ile gecildi ve QA otomation(functional) testler icin uygulamayi deploy eettim. Seleniumla 4 tane functional testlere tabi tuttum. Bunlar da test takiminin gorevi onlar tarafindan hazirlanmis olan testlere tabi tuttum. Buraya kadarki asamalari docker swarm ile yaptim ama bundan sonraki asamalari kubernetes ile gerceklestirdim. Daha sonra staging enviroment a gonderdim burada user acceptance ve QA testleri yaptim sonra productiona deployettim ve en sonunda prometheus ve grafana ile monintor edip projeyi bitirdim.

* Proje kapsaminda 5 tane pipeline olusturdum. 
  1. **Pipeline-ci-job:** dev, feature ve bugfix branchlari icin. Biri commit edildiginde jacoco ve Junit testleri yapildi. Commit icin git hesabimda olusturmus oldugum degisikler webhookla trigger edildi ve mavenla build teste tabi tutuldu ve birakildi. Bu pipeline bu asamaya kadar olacak. 
  2. **Pipeline-nightly:** dev branchinda yapilacak. Stabledir ama tum testlerden gecmemis bir versiyondur. Burada bir cron her gece 12 de dev branch inda kodu alacak testlere tabii tutacak ondan sonra functiona tutacak. Kullanilan toollar daha da artmaktadir.
  3. **Pipeline-weekly:** Release branchindaki kodu her hafta build edip deploy edecek bir pipeline olusturacagi test icin deploy edilecek bir pipeline olusturuledilmis kod manuel testerlarin onune dusucek onlar manuel testleri yapacaklar.
  4. **Pipeline-staging:** Haftalik olacak. Release brancinda olacak. Her hafta pazar gunu kodu alacak builde tabi tutacak. Build edecek, functional testleri yapacak sonra manuel testler icin manuel testerlara gidicek sonra staging env a gidecek bundan sonrasi artik kod ile ne yapilacagi ile ilgili. User acceptance testleri yapilabilir. Alfa surecine beta surecine gidilebilir veya kullanicinin musterinin kendi testlerini yapmak icin musteriye sunulabilir. Yani kodu staging env a birakacagiz. 
  5. **Pipeline-prod-webhook:** Master branchinda yapacagimiz her comitte bizim kodumuzu bastan alacak build, unit test ve productiona deploy edecek. Yani urunu musteriye sunacak ortama deploy edecek.

# Proje 29 taskdan olusmaktadir.

## Flow of Tasks for Project Realization

| Epic | Task  | Task #  | Task Definition   | Branch  |
| ---   | :---  | ---                  | :---              | :---    |
| Local Development Environment | Prepare Development Server Manually on EC2 Instance| MSP-1 | Prepare development server manually on Amazon Linux 2 for developers, enabled with `Docker` , `Docker-Compose` , `Java 11` , `Git` .  |
| Local Development Environment | Prepare GitHub Repository for the Project | MSP-2-1 | Clone the Petclinic app from the Clarusway repository [Petclinic Microservices Application](https://github.com/clarusway/petclinic-microservices.git) |
| Local Development Environment | Prepare GitHub Repository for the Project | MSP-2-2 | Prepare base branches namely `master` , `dev` , `release` for DevOps cycle. |
| Local Development Environment | Check the Maven Build Setup on Dev Branch | MSP-3 | Check the Maven builds for `test` , `package` , and `install` phases on `dev` branch |
| Local Development Environment | Prepare a Script for Packaging the Application | MSP-4 |  Prepare a script to package the application with Maven wrapper | feature/msp-4 |
| Local Development Environment | Prepare Development Server Cloudformation Template | MSP-5 |  Prepare development server script with Cloudformation template for developers, enabled with `Docker` , `Docker-Compose` , `Java 11` , `Git` . | feature/msp-5 |
| Local Development Build | Prepare Dockerfiles for Microservices | MSP-6 | | Prepare Dockerfiles for each microservices. | feature/msp-6 |
| Local Development Environment | Prepare Script for Building Docker Images | MSP-7 |  Prepare a script to package and build the docker images for all microservices. | feature/msp-7 |
| Local Development Build | Create Docker Compose File for Local Development | MSP-8-1 |  Prepare docker compose file to deploy the application locally. | feature/msp-8 |
| Local Development Build | Create Docker Compose File for Local Development | MSP-8-2 |  Prepare a script to test the deployment of the app locally. | feature/msp-8 |
| Testing Environment Setup | Implement Unit Tests | MSP-9-1  | Implement 3 Unit Tests locally. | feature/msp-9 |
| Testing Environment Setup | Setup Code Coverage Tool | MSP-9-2  | Update POM file for Code Coverage Report. | feature/msp-9 |
| Testing Environment Setup | Implement Code Coverage | MSP-9-3  | Generate Code Coverage Report manually. | feature/msp-9 |
| Testing Environment Setup | Prepare Selenium Tests | MSP-10-1  | Prepare 3 Selenium Jobs for QA Automation Tests. | feature/msp-10 |
| Testing Environment Setup | Implement Selenium Tests | MSP-10-2  | Run 3 Selenium Tests against local environment. | feature/msp-10 |
| CI Server Setup | Prepare Jenkins Server | MSP-11 | | Prepare Jenkins Server for CI/CD Pipeline. | feature/msp-11 |
| CI Server Setup | Configure Jenkins Server for Project | MSP-12  | Configure Jenkins Server for Project Setup. | |
| CI Server Setup | Prepare CI Pipeline | MSP-13 | | Prepare CI pipeline (UT only) for all `dev` , `feature` and `bugfix` branches. | feature/msp-13 |
| Registry Setup for Development | Create Docker Registry for Dev Manually | MSP-14  | Create Docker Registry on AWS ECR manually using Jenkins job. | |
| Registry Setup for Development | Prepare Script for Docker Registry| MSP-15  | Prepare a script to create Docker Registry on AWS ECR using Jenkins job. | feature/msp-15 |
| QA Automation Setup for Development | Create a QA Automation Environment | MSP-16  | Create a QA Automation Environment with Docker Swarm. | feature/msp-16 |
| QA Automation Setup for Development | Prepare a QA Automation Pipeline | MSP-17  | Prepare a QA Automation Pipeline on `dev` branch for Nightly Builds. | feature/msp-17 |
| QA Setup for Release | Create a Key Pair for QA Environment | MSP-18-1  | Create a permanent Key Pair for Ansible to be used in QA Environment on Release. | feature/msp-18 |
| QA Setup for Release | Create a QA Infrastructure with AWS Cloudformation | MSP-18-2  | Create a Permanent QA Infrastructure for Docker Swarm with AWS Cloudformation. | feature/msp-18 |
| QA Setup for Release | Create a QA Environment with Docker Swarm | MSP-18-3  | Create a QA Environment for Release with Docker Swarm. | feature/msp-18 |
| QA Setup for Release | Prepare Build Scripts for QA Environment | MSP-19  | Prepare Build Scripts for creating ECR Repo for QA, building QA Docker images, deploying app with Docker Compose. | feature/msp-19 |
| QA Setup for Release | Build and Deploy App on QA Environment Manually | MSP-20  | Build and Deploy App for QA Environment Manually using Jenkins Jobs. | feature/msp-20 | 
| QA Setup for Release | Prepare a QA Pipeline | MSP-21  | Prepare a QA Pipeline using Jenkins on `release` branch for Weekly Builds. | feature/msp-21 |
| Staging and Production Setup | Prepare Petlinic Kubernetes YAML Files | MSP-22  | Prepare Petlinic Kubernetes YAML Files for Staging and Production Pipelines. | feature/msp-22 |
| Staging and Production Setup | Prepare HA RKE Kubernetes Cluster | MSP-23  | Prepare High-availability RKE Kubernetes Cluster on AWS EC2 | feature/msp-23 |
| Staging and Production Setup | Install Rancher App on RKE K8s Cluster | MSP-24  | Install Rancher App on RKE Kubernetes Cluster| |
| Staging and Production Setup | Create Staging and Production Environment with Rancher | MSP-25  | Create Staging and Production Environment with Rancher by creating new cluster for Petclinic | |
| Staging Deployment Setup | Prepare a Staging Pipeline | MSP-26  | Prepare a Staging Pipeline on Jenkins Server | feature/msp-26|
| Production Deployment Setup | Prepare a Production Pipeline | MSP-27  | Prepare a Production Pipeline on Jenkins Server | feature/msp-27|
| Production Deployment Setup | Set Domain Name and TLS for Production | MSP-28  | Set Domain Name and TLS for Production Pipeline with Route 53 | feature/msp-28|
| Production Deployment Setup | Set Monitoring Tools | MSP-29  | Set Monitoring tools, Prometheus and Grafana | |

# MSP 1 - Prepare Development Server Manually on EC2 Instance

* Prepare development server manually on Amazon Linux 2 for developers, enabled with `Docker`,  `Docker-Compose`,  `Java 11`,  `Git`.

    • t2.Medium instance launch ettim.
    • Vscode ile instance a baglandim. Cunku cok fazla configurasyon dosyasi yazmam gerekiyor.
    • Localime petclinic-microservices reposunu dosyalarimi clone ettim.
    • GitHub dan repo olusturdum. Localimdeki dosyalari github repoma push ettim.
    • Ec2 ya gelerek github repomdaki dosyalarimi pull ettim.
    • Docker, Docker-compose, Git, java-11 kurulumunu yaptim.
Kullanilan userdata asagidaki gibidir:
```bash
#! /bin/bash
sudo yum update -y
sudo hostnamectl set-hostname petclinic-dev-server
sudo amazon-linux-extras install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user tn2/5376143
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" \
-o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
sudo yum install git -y
sudo yum install java-11-amazon-corretto -y
```

# 2. MSP 2 - Prepare GitHub Repository for the Project
## 2.1. Local Development Environment
Prepare GitHub Repository for the Project
MSP-2-1
Clone the Petclinic app from the Clarusway repository Petclinic Microservices Application

    • git configurasyon ayarlarini yaptim.
      ```bash
      git config --global user.name sezginerdem
      git config --global user.mail drsezginerdem@gmail.com
      git remote add origin https://github.com/sezginerdem/p01-petclinic-microservices-app.git
      ```
## 2.2.
Local Development Environment
Prepare GitHub Repository for the Project

MSP-2-2
Prepare base branches namely master , dev , release for DevOps cycle.

    • dev branch olusturdum.
      git branch dev
      git checkout dev
    • projemi dev branchina push ettim.
      git push -u origin dev
    • ayni islemleri release branchi icin yaptim.
      Git checkout master
      git branch release
      git checkout release
      git push -u origin release

# 3. MSP 3 - Check the Maven Build Setup on Dev Branch
Local Development Environment
Check the Maven Build Setup on Dev Branch
MSP-3
Check the Maven builds for test , package , and install phases on dev branch
      
    • Maven sadece bir build toolu. Maveni dockerfile a benzetebiliriz onun da kendine ait pom.xml (project object modal) adinda bir dosyasi var. Pom file maven icindeki proje hakkinda bilgileri iceriyor. Maven bu pom file I kullanarak bazi tasklar yapiyor. Bunlar goals diye adlandiriliyor. Bu goallere gore bazi stagelar var. Java ile maven ile kullandiginizda maven ince compile ediyor sonra tasklari kosuyor ama python interpreter bir dil olmadigi icin compile etmiyor. Tasklari kostuktan sonra executable bir jat file olusturuyor. Windows icin exe yi dusunebilirsiniz. Temel olarak javada kullaniliyor. Ama python icinde kullanildigi da olabiliyor ancak yaygin degil. Mvn –version komutu ile pc de oldugunu kontrol edebilirsin. Dedicate edilmis bir file structure i var. pom.xml icinde bu projenin ne olduguna dair tum dosyalar bulunuyor, dependiciesler neler, librariesler neler, test librariesler neler, github gibi mvn repository den build etmek icin requirementslari indiriyor. Bunlar java repository. Applicationi compile etmek ve packaging icin kullaniyor. Pom.xml icin temelde 2 dosyasi var src ve test folder. Src foler application a ait filelarin bulundugu folder. App.java application kod. Test file ise yalnizca test icin kullanilan kod. Application mvn test calistirildiginda test kodu icerisindeki kodlar calisiyor. Packaging yapildiginda mvn target adinda baska bir klasor olusturuyor. Bu target folder tum compile filelari kapsar. App.java yi compile ettigimizde app.class adinda bir dosya olusur bu bir compile koddur. Compile edilmeden once application icinde pek cok librariesler var fakat compile edildikten sonra target klasoru icinde .jar file ile birlikte tek bir executable file haline geliyor. Mvn create edildiginda tum dosya yapisini olusturuyor ve bu bos klasorlerin ici developerlar tarafindan dolduruluyor. Mvn clean calistirdiginizda bu komut onceden yapilan tum buildleri siliyor libraries, dependencies vs. mvn ile ilgili olan. Build lifecycles da ilk once validate ediliyor yani projeyi validate ediyor pom file I kontrol ediyor ve infolarin elde edilebilir olup olmadigini kontrol ediyor. Sonraki asama compile ediyor sonrasinda application code daki testleri yapiyor sonrasinda application I packaging yapiyor sonra verify asamasi var asamada integration testlerini kosuyor sonra install asamasi package i local repositorye install ediyor ve en son olarak deploy asamasi var localdeki install ettigim hepsini remote repositorye gonderiyor. Bu default lifecycle clean lifecycle daha farkli. 
      Mavenda pluginler onemli pluginlere farkli pluginler yukleyerek proje cesitlendirilebilir.
    • Projenin kendi pom.xml dosyasi oldugu gibi. Her microservisin kendi icinde pom.xml dosyasi var. microservislerin icinde bu gorulebilir. Su an icin herhangi bir target klasorumuz yok.
    • Pom file imiz var ama src file imiz yok. Bu su demek pom.xml dosyasini degistirerek tum yapiyi degistirebilirsiniz. Pom.xml bir semsiye olararak dusunebilirsiniz. Microservice ler altinda modul olarak duruyor. Target i yok. Moduller ise her biri sub maven project olarak duruyor. Mvn top pom.xml icin executable bir dosya olusturmuyor altindaki dosyalar icin target klasoru olsturuyor. 
    • Pom file mvn tarafindan ilk olarak otomatik olarak maven tarafindan yaratiliyor. Ama sonrasinda biz komut ile create ediyoruz.
      
    • Mvn i yuklemedik ama bash script ./mvnw komutunu calistirabiliyor. Kok klasorunun altinda pom.xml dosyasini otomatik olarak buluyor dockerfile da oldugu gibi sonra calistirabiliyor herhnagi bir yuklemeye gerek kalmiyor. 
    • Petclinic uygulamasinin testini yaptim.
      ```bash
      ./mvnw clean test
      ```
    • Permission denied erroru verdi. Bunun icin execute yetkisi verdim.
      ```bash
      chmod +x mvnw
      ```
    • Yeniden test komutunu girdim. Testlerimi yaptim.
      ```bash
      ./mvnw clean test
      ```
    • gerekli library leri mvn repodan indirmeye basladi. Ilk seferde uzun suruyor ama sonrasinda indirme olmadigi icin sonraki komutlarda bu kadar uzun surmeyecek. Bu komut ile validate compile ve test komutlarina kadar calisti. Sonraki asamalar calismadi. Yani executable bir dosya olusturmadi cunku packaging asamasina gecmedi bu komutla.
    • Bu test ile source kodumu derledim ve her bir source kodumun unit testlerini yaptim. Bu asamada her bir micro servisimin ayri ayri testlerini yaptim. Kodumun hatasiz oldugunu gormek icin bunu yaptim. Eger build etmese idim kodumun hatali olup olmadigini goremeyecektim. Source kodum unit testlerini basari ile gecti.
    • Uygulamayi paketlemek icin asagidaki kodu girdim. Uygulamami kurulmaya hazir bir .jar dosyasi haline getirdim. Clean komutu ile once mvn dosyalarini siliyor ondan sonra packaging yapiyor. Clean yapmak zorunda degilim cunku her sey tamam testlerden gecti sadece packaging yapabilirim ama bu asamalari gecmek istemiyorum yeniden yapmasini istiyorum. Bunu yapmak icin parametre girip degistirebilirim. 
      ```bash
      ./mvnw clean package
      ```
    • bu komuttan sonra ec2 home icinde .m2 dosyasi goruluyor. Bu dosya mvn in home u. 
    • mvnw dosyalarini siliyorum ve uygulamayi install ediyorum.
      ```bash
      ./mvnw clean install
      ```
      
# 4. MSP 4 - Prepare a Script for Packaging the Application      
Local Development Environment
Prepare a Script for Packaging the Application
MSP-4
Prepare a script to package the application with Maven wrapper
feature/msp-4
      
    • dev den feature branch olusturdum
      ```bash
      git checkout dev
      git branch feature/msp-4
      git checkout feature/msp-4
      ```
    • Kok dizini altinda package-with-mvn-wrapper.sh adinda script dosyasi olusturdum. Scriptin icine asagidaki komutu koydum.
      ```bash
      ./mvnw clean package
      ```

    • Commit ve push ettim.
      ```bash
      git add .
      git commit -m 'added packaging script'
      git push --set-upstream origin feature/msp-4
      git checkout dev
      git merge feature/msp-4
      git push origin dev
      ```
    • her zaman dev branch inda calisacagim icin dev branch ini default branch yapabilirim. Github imdan repository>settings>default barnch dev>update
# 5. MSP 5 - Prepare Development Server Cloudformation Template
      
Local Development Environment
Prepare Development Server Cloudformation Template
MSP-5
Prepare development server script with Cloudformation template for developers, enabled with Docker , Docker-Compose , Java 11 , Git .
feature/msp-5
      
    • dev den feature branch olusturdum
      ```bash
      git checkout dev
      git branch feature/msp-5
      git checkout feature/msp-5
      ```
    • Bundan onceki tum asamalari kapsayan dev-server-for-petclinic-app-cfn-template.yml adinda bir CF hazirladim. Bunu da kok dizinin altinda olusturdugum infrastructure klasorunun icine koydum. Burada userdatada kendi repomun ismimi yazmam onemli.

```yaml
AWSTemplateFormatVersion: 2010-09-09

Description: >
  This Cloudformation template prepares development environment for Petclinic Microservices Application.
  User needs to select appropriate key name when launching the template.
Parameters:
  KeyPairName:
    Description: Enter the name of your Key Pair for SSH connections.
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: Must one of the existing EC2 KeyPair

Resources:
  PetclinicDemoSG: 
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH and HTTP for Petclinic Microservices
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 9090
          ToPort: 9090
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 8080
          ToPort: 8080
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 8081
          ToPort: 8081
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 8082
          ToPort: 8082
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 8083
          ToPort: 8083
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 8888
          ToPort: 8888
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 9411
          ToPort: 9411
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 7979
          ToPort: 7979
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 3000
          ToPort: 3000
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 9091
          ToPort: 9091
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 8761
          ToPort: 8761
          CidrIp: 0.0.0.0/0
  PetclinicServerLT:
    Type: "AWS::EC2::LaunchTemplate"
    Properties:
      LaunchTemplateData:
        ImageId: ami-038b5c5d2ad6a5090
        InstanceType: t2.medium
        KeyName: !Ref KeyPairName
        SecurityGroupIds:
          - !GetAtt PetclinicDemoSG.GroupId
        UserData:
          Fn::Base64: |
            #! /bin/bash
            yum update -y
            hostnamectl set-hostname petclinic-dev-server
            amazon-linux-extras install docker -y
            systemctl start docker
            systemctl enable docker
            usermod -a -G docker ec2-user
            curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" \
            -o /usr/local/bin/docker-compose
            chmod +x /usr/local/bin/docker-compose
            yum install git -y
            yum install java-11-amazon-corretto -y
            git clone https://github.com/sezginerdem/p01-petclinic-microservices-app.git
            cd petclinic-microservices
            git fetch
            git checkout dev
  PetclinicServer:
    Type: AWS::EC2::Instance
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref PetclinicServerLT
        Version: !GetAtt PetclinicServerLT.LatestVersionNumber
      Tags:                
        - Key: Name
          Value: !Sub Petclinic App Dev Server of ${AWS::StackName}
Outputs:
  PetclinicServerDNSName:
    Description: Petclinic App URL
    Value: !GetAtt PetclinicServer.PublicDnsName
```

    • Commit ve push ettim.
      ```bash
      git add .
      git commit -m 'added cloudformation template for dev server'
      git push --set-upstream origin feature/msp-5
      git checkout dev
      git merge feature/msp-5
      git push origin dev
      ```
# 6. MSP 6 - Prepare Dockerfiles for Microservices
      
Local Development Build
Prepare Dockerfiles for Microservices
MSP-6

Prepare Dockerfiles for each microservices.
      
    • yeni feature yarattim
      ```bash
      git checkout dev
      git branch feature/msp-6
      git checkout feature/msp-6
      ```
    • Her bir microservisleri calistirmak icin bu microservisleri image haline getirmek icin bunlarin docker file larini olusturuyorum.
    • admin-server klasorunun altinda spring-petclinic-admin-server docker file ini olusturdum.
```Dockerfile
FROM openjdk:11-jre #neden java jdk degil de open jdk image cunku boyutu kucuk requirements larimi karsiliyor, bende maven oldugu icin javanin compilelarina ihtiyacim yok o da olunca imagein boyutu buyuyor. Biri 400 digeri 100 mb
ARG DOCKERIZE_VERSION=v0.6.1 #image olustururken build ederken kolaylik saglamak acisindan bir degisken atamak icin ARG komutunun kullandim. Version lar yenilenirken her seferinde ugrasmak istemedim
ARG EXPOSED_PORT=9090 #expose portunun degisme ihtimalina karsi bunu da arg icine koydum
ENV SPRING_PROFILES_ACTIVE docker #container olusurken icine env degiskeni eklemek icin yazdim. Developerlarin soyledigi env i koydum. Bunu da nerden bulabilirim. Customer service klasorunde bootstrap.yaml dosyasinin icinde port bilgisinde localhost dan degil de dockerdan alacagimiz bilgisi var. Bu da config serverda zaten bildirilmis ben de ona gore bu degiskeni atamam gerekiyor.
ADD https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz dockerize.tar.gz #github dan alpine linux dosyasini cekiyorum
RUN tar -xzf dockerize.tar.gz #tar klasorunu aciyor
RUN chmod +x dockerize #executable yetkisi verdim
ADD ./target/*.jar /app.jar #Mvn clean test yaptiktan sonra her bir micro serviste target klasoru olusacak ve onun icinde de .jar dosyalari olusacak. ./target dosyasi icerisinde ne kadar jar dosyasi varsa /app.jar dosyasinin icine at. 
EXPOSE ${EXPOSED_PORT} #hangi portu kullanacagimizi degisken olarak atadik
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"] #bu komutla container acildiginda hangi komutun calisacagini gosteriyoruz. CMD ile ENTRYPOINT arasindaki fark ise: cmd arguman alir yani calistirirken degistirebilirsin ama ENTRYPOINT de degistiremezsin. ENTRYPOINT de illaki koydugumuz komut calisacak demektir. Jar halien getirilms bir dosyayi calistirmak icin java -jar /app.jar komutu ile calistiriyoruz. "-Djava.security.egd=file:/dev/./urandom" bu komut java virtual machine in durmasini engelleyen ara katman.
```
    • Ayni dockerfile la api-gateway icindeki klasorde bir docker file olusturdum ve icerigin aynisini yapistirdim. Yalnizca port una baktim ve 8080 olarak degistirdim.
```Dockerfile      
FROM openjdk:11-jre
ARG DOCKERIZE_VERSION=v0.6.1
ARG EXPOSED_PORT=8080
ENV SPRING_PROFILES_ACTIVE docker
ADD https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz dockerize.tar.gz
RUN tar -xzf dockerize.tar.gz
RUN chmod +x dockerize
ADD ./target/*.jar /app.jar
EXPOSE ${EXPOSED_PORT}
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```      
    • Ayni dockerfile la config-server icindeki klasorde bir docker file olusturdum ve icerigin aynisini yapistirdim. Yalnizca port una baktim ve 8888 olarak degistirdim.
    • Ayni dockerfile la customer-service icindeki klasorde bir docker file olusturdum ve icerigin aynisini yapistirdim. Yalnizca port una baktim ve 8081 olarak degistirdim.
    • Ayni dockerfile la discovery-server icindeki klasorde bir docker file olusturdum ve icerigin aynisini yapistirdim. Yalnizca port una baktim ve 8761 olarak degistirdim.
    • Ayni dockerfile la hystrix-dashboard icindeki klasorde bir docker file olusturdum ve icerigin aynisini yapistirdim. Yalnizca port una baktim ve 7979 olarak degistirdim.
    • Ayni dockerfile la vets-service icindeki klasorde bir docker file olusturdum ve icerigin aynisini yapistirdim. Yalnizca port una baktim ve 8083 olarak degistirdim.
    • Ayni dockerfile la visit-service icindeki klasorde bir docker file olusturdum ve icerigin aynisini yapistirdim. Yalnizca port una baktim ve 8082 olarak degistirdim.
    • Degisiklikleri commit ettim ve push ettim.
```bash
git add .
git commit -m 'added Dockerfiles for microservices'
git push --set-upstream origin feature/msp-6
git checkout dev
git merge feature/msp-6
git push origin dev
```
# 7. MSP 7 - Prepare Script for Building Docker Images

Local Development Environment
Prepare Script for Building Docker Images
MSP-7
Prepare a script to package and build the docker images for all microservices.
feature/msp-7

Dockerfile lardan image olusturacagim bunun icin bir script yazacagim.
    • yeni feature yarattim
```bash
git checkout dev
git branch feature/msp-7
git checkout feature/msp-7
```
    • petclinic-microservices folder inin altinda build-dev-docker-images.sh scrip tini yazdim.
```bash
./mvnw clean package # tekrardan clean ve build yapmak icin bu komutu yazdim. Herhangi bir sey belki degismistir. Clean yaptim ve yeniden package ladim.
docker build --force-rm -t "petclinic-admin-server:dev" ./spring-petclinic-admin-server # --force-rm komutu build olustururken herhangi bir katmanda sorun olursa ya da basari ile build ederse tum katmanlari sil demek (boylece hafizadan tasarruf ediyorum). dev ile tagledim. Dev branch i bunu localde deneyecegi icin boyle tagledim. ./spring-petclinic-admin-server komutunu da dockerfile nerede ise orayi gosteren bir yol veriyorum. 
docker build --force-rm -t "petclinic-api-gateway:dev" ./spring-petclinic-api-gateway
docker build --force-rm -t "petclinic-config-server:dev" ./spring-petclinic-config-server
docker build --force-rm -t "petclinic-customers-service:dev" ./spring-petclinic-customers-service
docker build --force-rm -t "petclinic-discovery-server:dev" ./spring-petclinic-discovery-server
docker build --force-rm -t "petclinic-hystrix-dashboard:dev" ./spring-petclinic-hystrix-dashboard
docker build --force-rm -t "petclinic-vets-service:dev" ./spring-petclinic-vets-service
docker build --force-rm -t "petclinic-visits-service:dev" ./spring-petclinic-visits-service
docker build --force-rm -t "petclinic-grafana-server:dev" ./docker/grafana
docker build --force-rm -t "petclinic-prometheus-server:dev" ./docker/prometheus
```      
    • scriptime executable yetkisi verdim.
```bash
chmod +x build-dev-docker-images.sh
```
    • Scriptimi kontrol etmek icin scripti calistirdim. image lari build ettim. 
```bash
./build-dev-docker-images.sh
```
    • Commit ve push ettim.
```bash
git add .
git commit -m 'added script for building docker images'
git push --set-upstream origin feature/msp-7
git checkout dev
git merge feature/msp-7
git push origin dev
```
# 8. MSP 8 - Create Docker Compose File for Local Development
Programin kendisinde bir docker-compose file var ama ben onu kullanmayacagim. Imagelarin localde calismasi icin local imagelari kullanacagim. Benim tagledigim sekilde kullanacagim. Projede mevcut olan compose filedan cok az farkliliklar var. Bu sadece developerlarin kullanmasi icin olusturacagim bir docker-compose. Tek bir instance icin olusturacagim docker-compose file olusturdum. Docker-swarm icin kullanacagim docker-compose degil.
Local Development Build
Create Docker Compose File for Local Development
MSP-8-1
Prepare docker compose file to deploy the application locally.
feature/msp-8
      
    • yeni feature yarattim.
```bash
git checkout dev
git branch feature/msp-8
git checkout feature/msp-8
```
    • petclinic-microservices klasorunun icine docker-compose-local.yml adli bir file yarattim.
```yaml
Version: '2' #neden version 3 degilde 2: mem_limit kavrami version 3 de yok ama buna benim ihtiyacim var. mem_limit diyor ki bu image a verilen mb limiti. Boylelikle limit asimi oldugunda yalnizca o microservice cokecek tum app cokmeyecek. Version 3 de resources kaynagini koyup limiti eklemisler.

#Once config server calismasi lazim ki herkes configurasyon bilgilerini alsin. sonra service discovery calismasi lazim ki tum servisler bu servise “ben hazirim calisiyorum” diyebilsin. Bu ikisi calistiktan sonra program calismasi lazim.

services:
  config-server: #once config service olusturduk. Config service hicbirseye bagli degil.
    image: petclinic-config-server:dev
    container_name: config-server
    mem_limit: 512M 
    ports:
     - 8888:8888

  discovery-server:
    image: petclinic-discovery-server:dev
    container_name: discovery-server
    mem_limit: 512M
    depends_on: #config server calistiktan sonra calissin istiyorum
    - config-server
    entrypoint: ["./dockerize","-wait=tcp://config-server:8888","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"] #depends on her zaman yeterli degil. Servisin Calismasini bekliyor ama hazir olmasini beklemiyor. Bunun icin ilave programlara ihtiyaci var. bu dockerize komutu waiting for other depencies icin var. image lara ekledigimiz dockerize komutu burada isimize yaradi iste. -wait diyoruz neyi bekle config-serveri bekle. Ne kadar bekle 60s bekle. Calismazsa exit ver cik. Sonra yeniden calistir. Sonrasindaki “--” komutu ise bundan sonraki komutlar ayri komutlar yani java ya ait komutlar oldugunu belli ediyor.
    ports:
     - 8761:8761

  customers-service:
    image: petclinic-customers-service:dev
    container_name: customers-service
    mem_limit: 512M
    depends_on:
    - config-server
    - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"] #herseyi ayni sadece service ismi ve port numarasini degistirdim
    ports:
    - 8081:8081

  visits-service:
    image: petclinic-visits-service:dev
    container_name: visits-service
    mem_limit: 512M
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ports:
     - 8082:8082

  vets-service:
    image: petclinic-vets-service:dev
    container_name: vets-service
    mem_limit: 512M
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ports:
     - 8083:8083

  api-gateway:
    image: petclinic-api-gateway:dev
    container_name: api-gateway
    mem_limit: 512M
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ports:
     - 8080:8080

  tracing-server:
    image: openzipkin/zipkin #bunu kendimiz yapmadim bunu openzipkin in reposundan aldim. Bundan oncekilerin hepsini localimde olusturacagim oradan cekecegim ama bundan sonrakilerin hepsini dockerhub dan cekecegim
    container_name: tracing-server
    mem_limit: 512M
    environment:
    - JAVA_OPTS=-XX:+UnlockExperimentalVMOptions -Djava.security.egd=file:/dev/./urandom #Java 8 den sonra memory limitte sorun vardi. Memory sorununu cozmek icin buraya koydum
    ports:
     - 9411:9411

  admin-server:
    image: petclinic-admin-server:dev
    container_name: admin-server
    mem_limit: 512M
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ports:
     - 9090:9090

  hystrix-dashboard:
    image: petclinic-hystrix-dashboard:dev
    container_name: hystrix-dashboard
    mem_limit: 512M
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ports:
     - 7979:7979

  ## Grafana / Prometheus

  grafana-server:
    image: petclinic-grafana-server:dev
    container_name: grafana-server
    mem_limit: 256M
    ports:
    - 3000:3000

  prometheus-server:
    image: petclinic-prometheus-server:dev
    container_name: prometheus-server
    mem_limit: 256M
    ports:
    - 9091:9090 #prometheus un defaultu 9090 ama admin service in portu 9090 oldugu icin bunu 9091 den export etmeyi tercih ettim ayni olmasin diye
```

Local Development Build
Create Docker Compose File for Local Development
MSP-8-2
Prepare a script to test the deployment of the app locally.
feature/msp-8
    • docker-compose-local.yml adindaki dosyayi calistirmak icin test-local-deployment.sh scripti yaziyorum.
      ```bash
      docker-compose -f docker-compose-local.yml up #normalde docker compose up komutu ile calistirilir ama docker-compose file calismasin diye docker-compose-local.yml dosyasini calistirmak icin basina f koydum .
      ```
    • execute yetkisini veriyorum
      ```bash
      chmod +x test-local-deployment.sh
      ```
    • docker compose file ini calistiriyorum
      ```bash
      ./test-local-deployment.sh
      ```
    • Commit ve push ettim.
```bash      
git add .
git commit -m 'added docker-compose file and script for local deployment'
git push --set-upstream origin feature/msp-8
git checkout dev
git merge feature/msp-8
git push origin dev
```
# 9. MSP 9 - Setup Unit Tests and Configure Code Coverage Report
    • Bu asamada Unit test ve functional testleri implement ettim.
Testing Environment Setup
Implement Unit Tests
MSP-9-1
Implement 3 Unit Tests locally.
feature/msp-9
    • feature olusturdum.
```bash   
git checkout dev
git branch feature/msp-9
git checkout feature/msp-9
```
    • Unit test nedir: Birim test demek. Genelde kaynak kodu icerisindeki Fonksiyonlar unit test olarak belirlenir. En kucuk ve ilk test asamasi. Ilk olarak proje unit testi gectikten sonra build edilir.
    • Ornegin python sum fonksiyonu 5+6=11. Benim yazdigim fonksiyonun sum sonucu ile ayni olmasini bekleriz bununla ilgili bir test yazarim. Unit test developer sorumlulugunda. Source kod ile birlikte verir.
    • Customer-service icinde ./spring-petclinic-customers-service/src/test/java/org/springframework/samples/petclinic/customers/model/ icinde Pet.java isimli test dosyasi olusturdum. Icine de developer dan aldigim test code larini yazdim.
      
```json
package org.springframework.samples.petclinic.customers.model;

import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.Date;

import org.junit.jupiter.api.Test;
public class PetTest {
    @Test
    public void testGetName(){
        //Arrange
        Pet pet = new Pet();
        //Act
        pet.setName("Fluffy");
        //Assert
        assertEquals("Fluffy", pet.getName());
    }
    @Test
    public void testGetOwner(){
        //Arrange
        Pet pet = new Pet();
        Owner owner = new Owner();
        owner.setFirstName("Call");
        //Act
        pet.setOwner(owner);
        //Assert
        assertEquals("Call", pet.getOwner().getFirstName());
    }
    @Test
    public void testBirthDate(){
        //Arrange
        Pet pet = new Pet();
        Date bd = new Date();
        //Act
        pet.setBirthDate(bd);
        //Assert
        assertEquals(bd,pet.getBirthDate());
    }
}
```
    • Commit ettim.
```bash   
git add .
git commit -m 'added 3 UTs for customer-service'
git push --set-upstream origin feature/msp-9
```
    • Customer-service icinde
```bash 
. ../mvnw clean test
```
    • yukaridaki komut sorun verdi bu kodu girdim
```bash
source ../mvnw clean test
```
    • Jacoco bir code cocerage tool. Kodumun yuzde kacinin test edildigini gosteriyor.
Testing Environment Setup
Setup Code Coverage Tool
MSP-9-2
Update POM file for Code Coverage Report.
feature/msp-9

    • Kok klasordeki pom.xml dosyasinin 130 unda satirindan itibaren yani pluginslerden sonrasina bu kismi yapistiriyorum.
```js 
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.2</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <!-- attached to Maven test phase -->
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
    • Commit ve push ettim.
```bash      
git add .
git commit -m 'added 3 UTs for customer-service'
git push --set-upstream origin feature/msp-9
```

Testing Environment Setup
Implement Code Coverage
MSP-9-3
Generate Code Coverage Report manually.
feature/msp-9
    • Customer service klasorune girdim sadece jacoco plugin in calisip calismadigini kontrol etmek icin bu klasorun icinde asagidaki kodu calistirdim. Tum service ler icin yapmadim. Sadece jacoco nun calisip calismadigini kontrol ettim. Hepsini test etmeme gerek yok.
```bash
. ../mvnw test
```
    • target/site/jacoco klasorunun icine girdim. Buradaki html sayfasini calistirmak icin asagidaki kodu giriyorum ve html sayfasinda aciyorum. Jacoco raporu burada gorulebiliyor.
```bash
python -m SimpleHTTPServer # for python 2.7
python3 -m http.server # for python 3+
```
# 10. MSP 10 - Prepare and Implement Selenium Tests
      
Testing Environment Setup
Prepare Selenium Tests
MSP-10-1
Prepare 3 Selenium Jobs for QA Automation Tests.
feature/msp-10
    • Automatisation testleri test edicem seleniumla. Selenium testlerini hazirliyorum. Seleniumla manuel yapacak oldugum testleri otomatize edilmis haliyle kendi yapiyor hepsini. 
    • Feature yarattim.
      
      git checkout dev
      git branch feature/msp-10
      git checkout feature/msp-10

    • selenium-jobs adinda klasor olusturdum. test_owners_all_headless.py adinda bir dosya olusturdum. Test dosyasini icine kopyaladim.
```js
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from time import sleep
import os

# Set chrome options for working with headless mode (no screen)
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("headless")
chrome_options.add_argument("no-sandbox")
chrome_options.add_argument("disable-dev-shm-usage")

# Update webdriver instance of chrome-driver with adding chrome options
driver = webdriver.Chrome(options=chrome_options)

# Connect to the application
APP_IP = os.environ['MASTER_PUBLIC_IP']
url = "http://"+APP_IP.strip()+":8080/"
print(url)
driver.get(url)
sleep(3)
owners_link = driver.find_element_by_link_text("OWNERS")
owners_link.click()
sleep(2)
all_link = driver.find_element_by_link_text("ALL")
all_link.click()
sleep(2)

# Verify that table loaded
sleep(1)
verify_table = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.TAG_NAME, "table")))

print("Table loaded")

driver.quit()

    • ikinci testi test_owners_register_headless.py adi ile ekledim. Asagidaki test kodunu kopyaladim.

from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from time import sleep
import random
import os
# Set chrome options for working with headless mode (no screen)
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("headless")
chrome_options.add_argument("no-sandbox")
chrome_options.add_argument("disable-dev-shm-usage")

# Update webdriver instance of chrome-driver with adding chrome options
driver = webdriver.Chrome(options=chrome_options)

# Connect to the application
APP_IP = os.environ['MASTER_PUBLIC_IP']
url = "http://"+APP_IP.strip()+":8080/"
print(url)
driver.get(url)
owners_link = driver.find_element_by_link_text("OWNERS")
owners_link.click()
sleep(2)
all_link = driver.find_element_by_link_text("REGISTER")
all_link.click()
sleep(2)
# Register new Owner to Petclinic App
fn_field = driver.find_element_by_name('firstName')
fn = 'Callahan' + str(random.randint(0, 100))
fn_field.send_keys(fn)
sleep(1)
fn_field = driver.find_element_by_name('lastName')
fn_field.send_keys('Clarusway')
sleep(1)
fn_field = driver.find_element_by_name('address')
fn_field.send_keys('Ridge Corp. Street')
sleep(1)
fn_field = driver.find_element_by_name('city')
fn_field.send_keys('McLean')
sleep(1)
fn_field = driver.find_element_by_name('telephone')
fn_field.send_keys('+1230576803')
sleep(1)
fn_field.send_keys(Keys.ENTER)

# Wait 10 seconds to get updated Owner List
sleep(10)
# Verify that new user is added to Owner List
if fn in driver.page_source:
    print(fn, 'is added and found in the Owners Table')
    print("Test Passed")
else:
    print(fn, 'is not found in the Owners Table')
    print("Test Failed")
driver.quit()


    • test_veterinarians_headless.py adli dosyayi da ekledim asagidaki testi kopyaladim icine.

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from time import sleep
import os

# Set chrome options for working with headless mode (no screen)
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("headless")
chrome_options.add_argument("no-sandbox")
chrome_options.add_argument("disable-dev-shm-usage")

# Update webdriver instance of chrome-driver with adding chrome options
driver = webdriver.Chrome(options=chrome_options)

# Connect to the application
APP_IP = os.environ['MASTER_PUBLIC_IP']
url = "http://"+APP_IP.strip()+":8080/"
print(url)
driver.get(url)
sleep(3)
vet_link = driver.find_element_by_link_text("VETERINARIANS")
vet_link.click()

# Verify that table loaded
sleep(5)
verify_table = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.TAG_NAME, "table")))

print("Table loaded")

driver.quit()
```
    • commit ve push ettim.

git add .
git commit -m 'added selenium jobs written in python'
git push --set-upstream origin feature/msp-10
git checkout dev
git merge feature/msp-10
git push origin dev

    • Lokalde bu testleri calistirarak denedim.
Testing Environment Setup
Implement Selenium Tests
MSP-10-2
Run 3 Selenium Tests against local environment.
feature/msp-10

# 11. MSP 11 - Prepare Jenkins Server for CI/CD Pipeline
    • Jenkins serverimi olusturdum.

CI Server Setup
Prepare Jenkins Server
MSP-11

Prepare Jenkins Server for CI/CD Pipeline.
    • Feature olusturdum.
      
      git checkout dev
      git branch feature/msp-11
      git checkout feature/msp-11
      
    • Infrastructure folder inin icinde jenkins-server-cfn-template.yml adinda Jenkins server icin cloud formation olusturdum. 
```yaml
AWSTemplateFormatVersion: 2010-09-09

Description: >
  This Cloudformation Template creates a Jenkins Server using JDK 11 on EC2 Instance.
  Jenkins Server is enabled with Git, Docker and Docker Compose,
  AWS CLI Version 2, Python 3, Ansible, and Boto3. 
  Jenkins Server will run on Amazon Linux 2 EC2 Instance with
  custom security group allowing HTTP(80, 8080) and SSH (22) connections from anywhere.
Parameters:
  KeyPairName:
    Description: Enter the name of your Key Pair for SSH connections.
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: Must one of the existing EC2 KeyPair
Resources:
  EmpoweringRoleforJenkinsServer:
    Type: "AWS::IAM::Role" #ecr olusturacagim. Cloud formation islemi yapacagim icin rol yetkisi verdim
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              Service:
              - ec2.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess
        - arn:aws:iam::aws:policy/AWSCloudFormationFullAccess
        - arn:aws:iam::aws:policy/AdministratorAccess
  JenkinsServerEC2Profile:
    Type: "AWS::IAM::InstanceProfile"
    Properties:
      Roles: #required
        - !Ref EmpoweringRoleforJenkinsServer
  JenkinsServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH and HTTP for Jenkins Server
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 8080
          ToPort: 8080
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
  JenkinsServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0947d2ba12ee1ff75
      InstanceType: t2.medium
      KeyName: !Ref KeyPairName
      IamInstanceProfile: !Ref JenkinsServerEC2Profile
      SecurityGroupIds:
        - !GetAtt JenkinsServerSecurityGroup.GroupId
      Tags:                
        - Key: Name
          Value: !Sub Jenkins Server of ${AWS::StackName}
        - Key: server
          Value: jenkins
      UserData:
        Fn::Base64: |
          #! /bin/bash
          # update os
          yum update -y
          # set server hostname as jenkins-server
          hostnamectl set-hostname jenkins-server
          # install git
          yum install git -y
          # install java 11
          yum install java-11-amazon-corretto -y
          # install jenkins
          wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
          rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
          yum install jenkins -y
          systemctl start jenkins
          systemctl enable jenkins
          # install docker
          amazon-linux-extras install docker -y
          systemctl start docker
          systemctl enable docker
          usermod -a -G docker ec2-user
          usermod -a -G docker jenkins
          # configure docker as cloud agent for jenkins #dockeri agent olarak kullanicam kullandim maven testleri de bu docker agent araciligi ile yaptim. Normalde bunlari jenkins ve dev server olarak ikiya ayirabilirdim ancak maliyeti dusurmek icin tek makinada iki islemi de yapmayi tercih ettim. 
          cp /lib/systemd/system/docker.service /lib/systemd/system/docker.service.bak #lib/systemd/system/docker.service dosyasini /lib/systemd/system/docker.service.bak konumuna backup ini aliyorum
          sed -i 's/^ExecStart=.*/ExecStart=\/usr\/bin\/dockerd -H tcp:\/\/127.0.0.1:2375 -H unix:\/\/\/var\/run\/docker.sock/g' /lib/systemd/system/docker.service #sed -i komutu da stringleri degistirmek icin kullaniliyor. Burada da s/^ExecStart yerine .*/ExecStart=\/usr\/bin\/dockerd -H tcp:\/\/127.0.0.1:2375 -H unix:\/\/\/var\/run\/docker.sock/g bunu koy bunu da /lib/systemd/system/docker.service buranin icinde yap.  Docker a diyoruz ki localhost da 2375 portunu dinle oradan Jenkins sana talimat verecek onu dinle diyoruz. Bu Portu da ilerde tanimlayacagiz jenkins icinde
          systemctl daemon-reload
          systemctl restart docker
          systemctl restart jenkins
          # install docker compose
          curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" \
          -o /usr/local/bin/docker-compose
          chmod +x /usr/local/bin/docker-compose
          # uninstall aws cli version 1 #version 2 yi kurmak icin version 1 I kaldirmak gerekiyor
          rm -rf /bin/aws
          # install aws cli version 2
          curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
          unzip awscliv2.zip
          ./aws/install
          # install python 3
          yum install python3 -y
          # install ansible
          pip3 install ansible
          # install boto3
          pip3 install boto3
          
Outputs:
  JenkinsDNS:
    Description: Jenkins Server DNS Name 
    Value: !Sub 
      - ${PublicAddress}
      - PublicAddress: !GetAtt JenkinsServer.PublicDnsName
  JenkinsURL:
    Description: Jenkins Server URL
    Value: !Sub 
      - http://${PublicAddress}:8080
      - PublicAddress: !GetAtt JenkinsServer.PublicDnsName
```
    • CloudFormation u AWS ye yukledim. Stack imi create ettim.
    • SHH ile VSCode dan jenkise baglandim.
    • Arik bu instance dan devam edecegim icin git clone ile repomu buraya clone ladim.
    • Commit ledim ve push ettim.
      ```bash
      git add .
      git commit -m 'added jenkins server cfn template'
      git push --set-upstream origin feature/msp-11
      git checkout dev
      git merge feature/msp-11
      git push origin dev
      ```
# 12. MSP 12 - Configure Jenkins Server for Project
      
CI Server Setup
Configure Jenkins Server for Project
MSP-12
Configure Jenkins Server for Project Setup.
    • Jenkins Server imi configure ediyorum.
    • Jenkinsimin admin sifresini aliyorum.
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
    • Public IPnin devamini 8080 i yaziyorum ve http baglantisi ile jenkinsi aciyorum.
    • Sifremi girdim.
    • Tavsiye edilenPluginleri yukledim.
    • Admin user create ettim.
    • Dockeri cloud agent yapmak icin once docker plugin ve diger pluginleri yukluyorum: Manage jenkins>manage plugins>docker, docker pipeline, github integration, jacoco pluginlerini yukledim.
    • Dockeri agent olarak ayarlamak icin: manage jenkins>manage nodes and clouds>configure cloud>Add a new cloud>docker>docker cloud details>docker host url kismina tcp://localhost:2375 yazdim. 2375: docker kurulurken(user datadaki) yazdigim komuttaki port numarasi.
      
      13. MSP 13 - Prepare Continuous Integration (CI) Pipeline
      
CI Server Setup
Prepare CI Pipeline
MSP-13

Prepare CI pipeline (UT only) for all dev , feature and bugfix branches.
    • Projedeki ilk CI pipleline imi olusturdum.(WebHook)
    • Feature create ettim.
      
      git checkout dev
      git branch feature/msp-13
      git checkout feature/msp-13
      
    • Kok dizin altinda jenkins klasoru olusturdum.
      
      mkdir jenkins
      
    • Jenkins dashboarda geldim. new item>freestyle project adini petclinic-ci-job verdim. Source Code Management>Git>Githubdan project url yapistirdim. Branch to build>*/dev, */feature**, */bugfix** branch larini girdim. GitHub hook trigger for GITScm polling isaretledim. Add build step>execute shell kismina alttaki komutu girdim.
```bash
echo 'Running Unit Tests on Petclinic Application'
docker run --rm -v $HOME/.m2:/root/.m2 -v `pwd`:/app -w /app maven:3.6-openjdk-11 mvn clean test #maven 3.6 image ini calistir. Pwd deki yani jenkins in filelarin oldugu klasoru container icinde /app adli bir klasor olustur onun icine at. $HOME/.m2:/root/.m2 komutu ise: .m2 maven in local reposu demek. Jenkins in local reposuna indirdigi seyleri de jenkins in HOME undan container in /root/.m2 klasoru icine at ki bir daha her zaman bu containerdan kolay bir sekilde calistirabileyim. -v komutu volume olusturmak icin. -w ise calisma yerimizi gosteriyor. Workspce yani.
```
    • Post build actions>Add post-build action>Record jacoco coverage report ekledim. SAVE.
    • Build ettim. Outputu kontrol ettim.
    • Code Coverage I kontrol ettim. Jacoco plugin ile gorebiliyorum.
    • Webhook ayarlarini yapmak icin github a girdim. Settings>Webhooks>Add webhooks>Payload URL kismindaki public ip yi degistirdim.(http://[jenkins-server-hostname]:8080/github-webhook/) Add webhook tikladim.
    • Jenkins klasoru icine jenkins-petclinic-ci-job.sh olusturdum. Alttaki Scripti yazdim.
```bash
echo 'Running Unit Tests on Petclinic Application'
docker run --rm -v $HOME/.m2:/root/.m2 -v `pwd`:/app -w /app maven:3.6-openjdk-11 mvn clean test
```
    • commit ve push ettim.
      
      git add .
      git commit -m 'added Jenkins Job for CI pipeline'
      git push --set-upstream origin feature/msp-13
      git checkout dev
      git merge feature/msp-13
      git push origin dev
      
    • Webhook la trigger edildiginden pipelineim harekete gecti. Boylelikle developerlar bir kodu degistirdiklerinde trigger edecek ve pipeline calisacak.
      
      14. MSP 14 - Create Docker Registry for Dev Manually
      
Registry Setup for Development
Create Docker Registry for Dev Manually
MSP-14
Create Docker Registry on AWS ECR manually using Jenkins job.
    • Jenkins ile manuel olarak ECR yarattim. Amazonda calistigim icin image lari ecr da tutacagim dockerhub da degil.(EC2 yu kapatip acti isem webhook u guncellemem gerekir)
    • Jenkins dashboard a geldim. Newitem>freestyle project>create-ecr-docker-registry-for-dev adini yazdim OK bastim. Build step>execute shell kismina alttaki command I giriyorum.
```bash
PATH="$PATH:/usr/local/bin"
APP_REPO_NAME="sezgin-repo/petclinic-app-dev" #repo ismini degistirebilirsin
AWS_REGION="us-east-1"

aws ecr create-repository \
  --repository-name ${APP_REPO_NAME} \ #isim veriyorum
  --image-scanning-configuration scanOnPush=false \ #hassas durumlara karsi scan etmesini istemiyorum
  --image-tag-mutability MUTABLE \ #image lerden ayni isimde ikinci image olusturdugumuzda uzerine yazmaya izin vermiyor, hata veriyor.
  --region ${AWS_REGION} #region giriyorum
```
      15. MSP 15 - Prepare Script for Development Docker Registry
      
Registry Setup for Development
Prepare Script for Docker Registry
MSP-15
Prepare a script to create Docker Registry on AWS ECR using Jenkins job.
feature/msp-15
    • Docker registry olusturmak icin script olusturacagim.
    • Feature yarattim.
      
      git checkout dev
      git branch feature/msp-15
      git checkout feature/msp-15
      
    • Infrastructure altinda create-ecr-docker-registry-for-dev.sh adinda bir script yazdim.
```bash
PATH="$PATH:/usr/local/bin"
APP_REPO_NAME="sezgin-repo/petclinic-app-dev"
AWS_REGION="us-east-1"

aws ecr create-repository \
  --repository-name ${APP_REPO_NAME} \
  --image-scanning-configuration scanOnPush=false \
  --image-tag-mutability MUTABLE \
  --region ${AWS_REGION}
```
    • Commit ve push.
      
      git add .
      git commit -m 'added script for creating ECR registry for dev'
      git push --set-upstream origin feature/msp-15
      git checkout dev
      git merge feature/msp-15
      git push origin dev

# 16. MSP 16 - Create a QA Automation Environment with Docker Swarm

QA Automation Setup for Development
Create a QA Automation Environment
MSP-16
Create a QA Automation Environment with Docker Swarm.
feature/msp-16
    • Projedeki 2. pipeline i olusturmak istiyorum. Functional testleri yapmak icin docker swarm environment ini kurdum. Bunun icin 5 tane instance kullandim. 3 manager 2 worker in bu uygulama icin uygun oldugunu dusundum. 
    • feature olusturdum.
      
      git checkout dev
      git branch feature/msp-16
      git checkout feature/msp-16
      
    • infrastructure klasorunun icinde docker-swarm-infrastructure-cfn-template.yml olusturdumm. Instance larda docker, docker swarm vs yok. Bunlarin hepsini ansible ile olusturdum.
```yaml
AWSTemplateFormatVersion: 2010-09-09

Description: >
  This Cloudformation Template creates an infrastructure for Docker Swarm
  with five EC2 Instances with Amazon Linux 2. Instances are configured
  with custom security group allowing SSH (22), HTTP (80) UDP (4789, 7946), 
  and TCP(2377, 7946, 8080) connections from anywhere.
  User needs to select appropriate key name when launching the template.
Parameters:
  KeyPairName:
    Description: Enter the name of your Key Pair for SSH connections.
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: Must one of the existing EC2 KeyPair

Resources:  
  RoleEnablingEC2forECR: #her bir instance a bu rolu verdim. Ecr full access rolunu atiyorum.
    Type: "AWS::IAM::Role"
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              Service:
              - ec2.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess
  EC2Profile:
    Type: "AWS::IAM::InstanceProfile"
    Properties:
      Roles: #required
        - !Ref RoleEnablingEC2forECR
  DockerMachinesSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH and HTTP for Docker Machines
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 2377
          ToPort: 2377
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 7946
          ToPort: 7946
          CidrIp: 0.0.0.0/0
        - IpProtocol: udp
          FromPort: 7946
          ToPort: 7946
          CidrIp: 0.0.0.0/0
        - IpProtocol: udp
          FromPort: 4789
          ToPort: 4789
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 8080
          ToPort: 8080
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 8088
          ToPort: 8088
          CidrIp: 0.0.0.0/0
  DockerMachineLT:
    Type: "AWS::EC2::LaunchTemplate"
    Properties:
      LaunchTemplateData:
        ImageId: ami-0947d2ba12ee1ff75
        InstanceType: t2.medium
        KeyName: !Ref KeyPairName
        IamInstanceProfile: 
          Arn: !GetAtt EC2Profile.Arn
        SecurityGroupIds:
          - !GetAtt DockerMachinesSecurityGroup.GroupId
        TagSpecifications: #bu taglerle ansible icin dinamik inventory olusturacak bu nedenle onemli.
          - ResourceType: instance
            Tags: 
              - Key: app-stack-name
                Value: !Sub ${AWS::StackName}
              - Key: environment
                Value: dev
  DockerInstance1:
    Type: AWS::EC2::Instance
    DependsOn: #buna depends on yazdim cunku en son dockerinstance1 in gelmesini istiyorum. Ileri testleri bunun uzerinden yapacagim bu nedenle burada depends on var. 
        - "DockerInstance2"
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref DockerMachineLT
        Version: !GetAtt DockerMachineLT.LatestVersionNumber
      Tags:                
        - Key: server
          Value: docker-instance-1                       
        - Key: swarm-role
          Value: grand-master #manager lardan birisi grand-maste olmasi gerekiyordu bu instance i grand master olarak tagledim.
        - Key: Name
          Value: !Sub ${AWS::StackName} Docker Machine 1st        
  DockerInstance2:
    Type: AWS::EC2::Instance
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref DockerMachineLT
        Version: !GetAtt DockerMachineLT.LatestVersionNumber
      Tags:  #Ansible da dynamic inventory olusturmak icin bu kadar tag atadim
        - Key: server
          Value: docker-instance-2                       
        - Key: swarm-role
          Value: manager                       
        - Key: Name
          Value: !Sub ${AWS::StackName} Docker Machine 2nd
  DockerInstance3:
    Type: AWS::EC2::Instance
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref DockerMachineLT
        Version: !GetAtt DockerMachineLT.LatestVersionNumber
      Tags:                
        - Key: server
          Value: docker-instance-3                       
        - Key: swarm-role
          Value: manager                       
        - Key: Name
          Value: !Sub ${AWS::StackName} Docker Machine 3rd
  DockerInstance4:
    Type: AWS::EC2::Instance
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref DockerMachineLT
        Version: !GetAtt DockerMachineLT.LatestVersionNumber
      Tags:                
        - Key: server
          Value: docker-instance-4                       
        - Key: swarm-role
          Value: worker                       
        - Key: Name
          Value: !Sub ${AWS::StackName} Docker Machine 4th
  DockerInstance5:
    Type: AWS::EC2::Instance
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref DockerMachineLT
        Version: !GetAtt DockerMachineLT.LatestVersionNumber
      Tags:                
        - Key: server
          Value: docker-instance-5                       
        - Key: swarm-role
          Value: worker                       
        - Key: Name
          Value: !Sub ${AWS::StackName} Docker Machine 5th
Outputs:
  1stDockerInstanceDNSName:
    Description: 1st Docker Instance DNS Name
    Value: !Sub 
      - ${PublicAddress}
      - PublicAddress: !GetAtt DockerInstance1.PublicDnsName
  2ndDockerInstanceDNSName:
    Description: 2nd Docker Instance DNS Name
    Value: !Sub 
      - ${PublicAddress}
      - PublicAddress: !GetAtt DockerInstance2.PublicDnsName
  3rdDockerInstanceDNSName:
    Description: 3rd Docker Instance DNS Name
    Value: !Sub 
      - ${PublicAddress}
      - PublicAddress: !GetAtt DockerInstance3.PublicDnsName
  4thDockerInstanceDNSName:
    Description: 4th Docker Instance DNS Name
    Value: !Sub 
      - ${PublicAddress}
      - PublicAddress: !GetAtt DockerInstance4.PublicDnsName
  5thDockerInstanceDNSName:
    Description: 5th Docker Instance DNS Name
    Value: !Sub 
      - ${PublicAddress}
      - PublicAddress: !GetAtt DockerInstance5.PublicDnsName
```
    • Commit ve push
      
      git add .
      git commit -m 'added cloudformation template for Docker Swarm infrastructure'
      git push --set-upstream origin feature/msp-16
      
    • Oncelikle yapacagim automation oncesi command larimin calisip calismadigini test etmek icin teker teker freestyle project olarak manuel bicimde denemesini yapacagim.
    • Jenkins dashboard>new item>freestyle project>test-creating-qa-automation-infrastructure adini verdim OK. Source Code Management>Git>Project URL yapistir. Branches to Build>*/feature/msp-16 yazdim. Add build step>Execute Shell>alttaki command yapistirdim. SAVE.
```bash
echo $PATH #binarymi, command larimin oldugu klasoru goster
whoami #ben kimim(yani bu makine ec2 degil jenkins makinasi user olarak o cikiyor)
PATH="$PATH:/usr/local/bin" #aws cli calismasi icin path ekliyorum
python3 --version
pip3 --version
ansible --version
aws --version
```
    • Build yaptim. Output u kontrol ettim. Evet komutlarim calisiyor. Sorun olmadigini anlamis oldum.
    • Dasboard>test-creating-qa-automation-infrastructure>configure>execute shell deki commad in yerine asagidaki command i yaziyorum.
```bash
PATH="$PATH:/usr/local/bin" 
CFN_KEYPAIR="sezgin-ansible-test-dev.key" #keypair olusturuyorum
AWS_REGION="us-east-1" #hangi bolgede olusturacak
aws ec2 create-key-pair --region ${AWS_REGION} --key-name ${CFN_KEYPAIR} --query "KeyMaterial" --output text > ${CFN_KEYPAIR} #query komutu keyfile icerisindeki tum yazanlari al ve output text komutu ile benim keypair sezgin-ansible-test-dev.key imin icine koy. Boylece local klasorumde olusuyor.
chmod 400 ${CFN_KEYPAIR} #chmod yetkisi verdim.
```
    • Build ettim. Workspace den kontrol ettim. Pem key olusmus. Amazon Console dan da bakilirsa olustugunu gorebiliyorum.
    • Dasboard>test-creating-qa-automation-infrastructure>configure>execute shell deki commad in yerine asagidaki command i yaziyorum. SAVE.
```bash      
PATH="$PATH:/usr/local/bin"
APP_NAME="Petclinic"
APP_STACK_NAME="sezgin-$APP_NAME-App-${BUILD_NUMBER}"
CFN_KEYPAIR="sezgin-ansible-test-dev.key"
CFN_TEMPLATE="./infrastructure/dev-docker-swarm-infrastructure-cfn-template.yml" #template i buradan al dedim
AWS_REGION="us-east-1"
aws cloudformation create-stack --region ${AWS_REGION} --stack-name ${APP_STACK_NAME} --capabilities CAPABILITY_IAM --template-body file://${CFN_TEMPLATE} --parameters ParameterKey=KeyPairName,ParameterValue=${CFN_KEYPAIR} #aws cloud formation create-stack=stack olustur command. --region hangi region da olusacagi. --stack-name= adinin ne olacagi. --capabilities=rol tanimlamasini acikca yapmamizi sagliyor(consoledan cf olustururken IAM role tanimlamasi yapmak istiyor musun diye soruyor iste bu command ona denk geliyor). --template-body file=template nereden olusturulacak klasoru yazdim. --parameters=parameters degerine de olusturdugum CFN_KEYPAIR i koydum.
```
    • Build. AWS Cloud formation a gelip baktigimda stack numarasinin otomatik atandigi gorulebiliyor. AWS Dashboard a geldigimde 5 tane ec2 nun olustugunu gorebiliyorum.
    • Jenkins e SSH ile  baglanabiliyor muyum onu kontrol etmek istiyorum. Dasboard>test-creating-qa-automation-infrastructure>configure>execute shell deki commad in yerine asagidaki command I yaziyorum. SAVE.
```bash
CFN_KEYPAIR="sezgin-ansible-test-dev.key"
ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -i ${WORKSPACE}/${CFN_KEYPAIR} ec2-user@172.31.91.243 hostname #ip degistir.  her seferinde degistirmemek icin Private IP yaptim. -o strictHostKeyChecking=no commandi giriste bize yes mi sorusu sormamasi icin kullandim. -o UserKnownHostsFile=/dev/null command i normalde .ssh klasorunu icinde baglanilan instance lari known_host file inin altina kaydediyor, bazen bunu kaydedeyim diye yes mi no mu diye soruyor. Bu command ile bunu bosa at bana sorma demek istedim. Bir nevi topraklama.
```
    • Ansible icin statik bir inventory olusturmak icin: ansible/inventory adli bir klasor olusturdum. Altina hosts.ini adinda bir file olusturdum. Docker machinelerin private ip lerini koydum. 
```txt
172.31.91.243   ansible_user=ec2-user  
172.31.87.143   ansible_user=ec2-user
172.31.90.30    ansible_user=ec2-user
172.31.92.190   ansible_user=ec2-user
172.31.88.8     ansible_user=ec2-user
```
    • Commit ve push ettim.
      
      git add .
      git commit -m 'added ansible static inventory host.ini for testing'
      git push
      
    • Dasboard>test-creating-qa-automation-infrastructure>configure>execute shell deki commad in yerine asagidaki command i yaziyorum. SAVE.
```bash
PATH="$PATH:/usr/local/bin"
CFN_KEYPAIR="sezgin-ansible-test-dev.key"
export ANSIBLE_INVENTORY="${WORKSPACE}/ansible/inventory/hosts.ini" #export command i ile ANSIBLE_INVENTORY nun icine ${WORKSPACE}/ansible/inventory/hosts.ini dosyasini koy
export ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${CFN_KEYPAIR}" #keyfile i alip  ANSIBLE_PRIVATE_KEY_FILE atamasini yaptim.
export ANSIBLE_HOST_KEY_CHECKING=False #host key her seferinde sormamasi icin
ansible all -m ping #ping atiyorum ki calisiyor mu diye
```
    • ansible inventory nin baktigi dosya sirasi yanda, ben yukaridaki command ile ona hosts.inin yerini gosterdim.
    • Build ettim. Sorunsuz calistigi icin ansible a sorunsuz baglandi.
    • Ansible da dinamik inventory olusturmak icin ansible/inventory dosyasinin icine dev_stack_dynamic_inventory_aws_ec2.yaml dosyasi olusturdum. 

```yaml
plugin: aws_ec2
regions:
  - "us-east-1"
filters: #filtrele tag i dev ve APP_STACK_NAME olanlari filtrele.
  tag:app-stack-name: APP_STACK_NAME 
  tag:environment: dev
keyed_groups: #grupluyorum
  - key: tags['app-stack-name'] #app-stack-name i al
    prefix: 'app_stack_' #aldigim bu app-stack-nameli instancelarin basina app_stack koy
    separator: ''
  - key: tags['swarm-role'] #swarmdaki rollerine gore gruplandir
    prefix: 'role_' #basina role_ koy
    separator: ''
  - key: tags['environment'] #environment olanlari bul
    prefix: 'env_' #basina env_ ekle
    separator: ''
  - key: tags['server'] #server olarak gruplandir
    separator: ''
hostnames:
  - "private-ip-address"
compose:
  ansible_user: "'ec2-user'"
```
    • Ansible da dinamik inventory olusturmak icin ansible/inventory dosyasinin icine dev_stack_swarm_grand_master_aws_ec2.yaml dosyasi olusturdum. 

```yaml
plugin: aws_ec2
regions:
  - "us-east-1"
filters: #alttaki 3 tag i olan instance i filtrele, onun ip adresini al.
  tag:app-stack-name: APP_STACK_NAME
  tag:environment: dev
  tag:swarm-role: grand-master
hostnames:
  - "private-ip-address"
compose:
  ansible_user: "'ec2-user'"
```
    • Ansible da dinamik inventory olusturmak icin ansible/inventory dosyasinin icine dev_stack_swarm_managers_aws_ec2.yaml dosyasi olusturdum. 
```yaml
plugin: aws_ec2
regions:
  - "us-east-1"
filters: #yukaridakinin ayni mantik gecerli. Manager lari grupladik ip adreslerini aldik
  tag:app-stack-name: APP_STACK_NAME
  tag:environment: dev
  tag:swarm-role: manager
hostnames:
  - "private-ip-address"
compose:
  ansible_user: "'ec2-user'"
```
    • Ansible da dinamik inventory olusturmak icin ansible/inventory dosyasinin icine dev_stack_swarm_workers_aws_ec2.yaml dosyasi olusturdum. 
```yaml
plugin: aws_ec2
regions:
  - "us-east-1"
filters: #yukaridakinin ayni mantik gecerli. Manager lari grupladik ip adreslerini aldik
  tag:app-stack-name: APP_STACK_NAME
  tag:environment: dev
  tag:swarm-role: worker
hostnames:
  - "private-ip-address"
compose:
  ansible_user: "'ec2-user'"
```
    • Commit ve push.

git add .
git commit -m 'added ansible dynamic inventory files for dev environment'
git push

    • Jenkins>Dashboard>test-creating-qa-automation-infrastructure>configure>command i degistir.
```bash
APP_NAME="Petclinic"
CFN_KEYPAIR="sezgin-ansible-test-dev.key"
PATH="$PATH:/usr/local/bin"
export ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${CFN_KEYPAIR}"
export ANSIBLE_HOST_KEY_CHECKING=False
export APP_STACK_NAME="sezgin-$APP_NAME-App-${BUILD_NUMBER}" #build number aws console cloud formation dan stack numrasinin aynisi yazilacak
# Dev Stack
sed -i "s/APP_STACK_NAME/$APP_STACK_NAME/" ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml #./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml APP_STACK_NAME adini yukarida tanimladigim APP_STACK_NAME I yaz.
cat ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml
ansible-inventory -v -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml –graph #ansible inventory -v -i commandi olusturdgumuz inventory dosyasini calistiriyor. 
# Dev Stack Grand Master
sed -i "s/APP_STACK_NAME/$APP_STACK_NAME/" ./ansible/inventory/dev_stack_swarm_grand_master_aws_ec2.yaml
cat ./ansible/inventory/dev_stack_swarm_grand_master_aws_ec2.yaml
ansible-inventory -v -i ./ansible/inventory/dev_stack_swarm_grand_master_aws_ec2.yaml --graph
# Dev Stack Managers
sed -i "s/APP_STACK_NAME/$APP_STACK_NAME/" ./ansible/inventory/dev_stack_swarm_managers_aws_ec2.yaml
cat ./ansible/inventory/dev_stack_swarm_managers_aws_ec2.yaml
ansible-inventory -v -i ./ansible/inventory/dev_stack_swarm_managers_aws_ec2.yaml --graph
# Dev Stack Workers
sed -i "s/APP_STACK_NAME/$APP_STACK_NAME/" ./ansible/inventory/dev_stack_swarm_workers_aws_ec2.yaml
cat ./ansible/inventory/dev_stack_swarm_workers_aws_ec2.yaml
ansible-inventory -v -i ./ansible/inventory/dev_stack_swarm_workers_aws_ec2.yaml –graph
```
    • Build ettim.
    • Ping atarak kontrol ediyorum. Jenkins>Dashboard>test-creating-qa-automation-infrastructure>configure>command i degistir.
```bash
# Test dev dynamic inventory by pinging
APP_NAME="Petclinic"
CFN_KEYPAIR="sezgin-ansible-test-dev.key"
PATH="$PATH:/usr/local/bin"
export ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${CFN_KEYPAIR}"
export ANSIBLE_HOST_KEY_CHECKING=False
export APP_STACK_NAME="Sezgin-$APP_NAME-App-${BUILD_NUMBER}" #build numberi degistirmem gerekir. Sabit olarak stack numarasina bakip onu yazmam gerek
sed -i "s/APP_STACK_NAME/$APP_STACK_NAME/" ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml
ansible -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml all -m ping
```
    • Build ettim. Dynamic inventory olustu.
    • ansible/playbooks folder olusturdum. pb_setup_for_all_docker_swarm_instances.yaml isimli yaml dosyasini koydum. Icerigi asagida (cf deki user data da yapmis oldugum islemleri yapiyorum)
```yaml
---
- hosts: all #tum instance lara ayni islemi yap
  tasks:
  - name: update os #guncel hale getir
    yum: #bu bir modul
      name: '*'
      state: present #update os buraya kadar devam eden komut
  - name: install docker #docker install ettim
    command: amazon-linux-extras install docker=latest -y
  - name: start docker
    service:
      name: docker
      state: started #start ettim
      enabled: yes
  - name: add ec2-user to docker group #ec2 yu docker gruba ekledim
    shell: "usermod -a -G docker ec2-user" #dockeri sudosuz calistirmak icin bu command gerekli
  - name: install docker compose. #docker compose yukle
    get_url:
      url: https://github.com/docker/compose/releases/download/1.26.2/docker-compose-Linux-x86_64 
      dest: /usr/local/bin/docker-compose
      mode: 0755
  - name: uninstall aws cli v1 # cli1 uninstall
    file:
      path: /bin/aws #path gosterdim
      state: absent
  - name: download awscliv2 installer #cli2 yukle
    unarchive:
      src: https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip
      dest: /tmp #aws cli al tmp e koy
      remote_src: yes
      creates: /tmp/aws #tmp/aws ye koy
      mode: 0755
  - name: run the installer
    command:
    args:
      cmd: "/tmp/aws/install" #chmod yap
      creates: /usr/local/bin/aws #calistir
```
    • ansible/playbooks folder olusturdum. pb_initialize_docker_swarm.yaml isimli yaml dosyasini koydum. Icerigi asagida
```yaml
---
- hosts: role_grand_master #docker swami initialize ettim
  tasks:
  - name: initialize docker swarm #docker swarm yuklendi
    shell: docker swarm init
  - name: install git #git control edildi en son versiyonu yuklendi
    yum:
      name: git
      state: present
  - name: run the visualizer app for docker swarm #visualizer yuklendi
    shell: |
      docker service create \
        --name=viz \
        --publish=8088:8080/tcp \
        --constraint=node.role==manager \
        --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
        dockersamples/visualizer
```
    • ansible/playbooks folder olusturdum. pb_join_docker_swarm_managers.yaml isimli yaml dosyasini koydum. Icerigi asagida
```yaml
---
- hosts: role_grand_master #manager ekledim
  tasks:
  - name: Get swarm join-token for managers #token aliyorum
    shell: docker swarm join-token manager | grep -i 'docker' #sadece docker yazan satiri almak istedigim icin grep -i docker komutunu yazdim
    register: join_command_for_managers #bir ustteki cikan komutu join_command_for_managers olarak yazdiriyorum

  - debug: msg='{{ join_command_for_managers.stdout.strip() }}' #yazilan komutta sadece standart output istedigim icin stdout.strip() komutunu ekliyorum
  
  - name: register grand_master with variable
    add_host: #bu modul ayni playbook icerisinde bir playden diger playa degisken atamak icin kullanilir. Yukaridaki bir playden asagidaki bir playe komutu aktarmak icin kullaniyorum. Cache Memory de tutuyor 
      name: "grand_master" # isim verdim ve manager join olarak komutu tutmak icin kaydettim
      manager_join: "{{ join_command_for_managers.stdout.strip() }}"

- hosts: role_manager #manager dan aldigim komutu worker a aktariyorum
  tasks:
  - name: Join managers to swarm
    shell: "{{ hostvars['grand_master']['manager_join'] }}" #grandmaster I playbook icerisinde buluyor ve buna bagli maneger join komutunu burada calistiriyor.
    register: result_of_joining

- debug: msg='{{ result_of_joining.stdout }}' #bu komutla da son calistirdigim komutu gormek icin yaziyorum. Son komutu gosteriyor. Bir nevi console log ciktisi veriyor.
```
    • ansible/playbooks folder altina. pb_join_docker_swarm_managers.yaml isimli yaml dosyasini koydum. Icerigi asagida
```yaml
---
- hosts: role_grand_master #grand master daki ayni islemi burada yapiyorum sadece tek farki worker olmasi
  tasks:
  - name: Get swarm join-token for workers
    shell: docker swarm join-token worker | grep -i 'docker'
    register: join_command_for_workers

  - debug: msg='{{ join_command_for_workers.stdout.strip() }}'
  
  - name: register grand_master with variable
    add_host:
      name: "grand_master"
      worker_join: "{{ join_command_for_workers.stdout.strip() }}"

- hosts: role_worker
  tasks:
  - name: Join workers to swarm
    shell: "{{ hostvars['grand_master']['worker_join'] }}"
    register: result_of_joining

  - debug: msg='{{ result_of_joining.stdout }}'
```
    • commit ve push.
      
      git add .
      git commit -m 'added ansible playbooks for dev environment'
      git push
      
    • Jenkins>Dashboard>test-creating-qa-automation-infrastructure>configure>command I degistir.
```bash
APP_NAME="Petclinic"
CFN_KEYPAIR="sezgin-ansible-test-dev.key"
PATH="$PATH:/usr/local/bin"
export ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${CFN_KEYPAIR}"
export ANSIBLE_HOST_KEY_CHECKING=False
export APP_STACK_NAME="sezgin-$APP_NAME-App-${BUILD_NUMBER}" #build numberi degistir
sed -i "s/APP_STACK_NAME/$APP_STACK_NAME/" ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml
# Swarm Setup for all nodes (instances) #sadece dev_stacki calistirdim digerleri egitim kapsaminda idi projede gerekli degildi. Bu komutla cf userdatayi bu playbook araciligi ile yaptim.
ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_setup_for_all_docker_swarm_instances.yaml #-i inventory den al demek. -b ise bu islemleri sudo yetkisi ile yap demek
# Swarm Setup for Grand Master node #docker swarmi initialize ettim.
ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_initialize_docker_swarm.yaml
# Swarm Setup for Other Managers nodes
ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_join_docker_swarm_managers.yaml
# Swarm Setup for Workers nodes
ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_join_docker_swarm_workers.yaml
```
    • Build ettim.
    • Olusturdugum Cloud Formationu siliyorum. 5 tane instance sildim. Jenkins>Dashboard>test-creating-qa-automation-infrastructure>configure>command i degistir.
```bash
PATH="$PATH:/usr/local/bin"
APP_NAME="Petclinic"
AWS_STACK_NAME="sezgin-$APP_NAME-App-${BUILD_NUMBER}" #Build number degistir
AWS_REGION="us-east-1"
aws cloudformation delete-stack --region ${AWS_REGION} --stack-name ${AWS_STACK_NAME}
```
    • Infrastructure klasoru icinde create-qa-automation-environment.sh adinda bir file olusturdum.
```bash
PATH="$PATH:/usr/local/bin"
APP_NAME="Petclinic"
CFN_KEYPAIR="sezgin-$APP_NAME-dev-${BUILD_NUMBER}.key"
CFN_TEMPLATE="./infrastructure/dev-docker-swarm-infrastructure-cfn-template.yml"
AWS_REGION="us-east-1"
export ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${CFN_KEYPAIR}"
export ANSIBLE_HOST_KEY_CHECKING=False
export APP_STACK_NAME="sezgin-$APP_NAME-App-${BUILD_NUMBER}"
# Create key pair for Ansible
aws ec2 create-key-pair --region ${AWS_REGION} --key-name ${CFN_KEYPAIR} --query "KeyMaterial" --output text > ${CFN_KEYPAIR}
chmod 400 ${CFN_KEYPAIR}
# Create infrastructure for Docker Swarm
aws cloudformation create-stack --region ${AWS_REGION} --stack-name ${APP_STACK_NAME} --capabilities CAPABILITY_IAM --template-body file://${CFN_TEMPLATE} --parameters ParameterKey=KeyPairName,ParameterValue=${CFN_KEYPAIR}

# Install Docker Swarm environment on the infrastructure
# Update dynamic inventory (hosts/docker nodes)
sed -i "s/APP_STACK_NAME/$APP_STACK_NAME/" ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml
# Install common tools on all instances/nodes
ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_setup_for_all_docker_swarm_instances.yaml
# Initialize Docker Swarm on Grand Master
ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_initialize_docker_swarm.yaml
# Join the manager instances to the Swarm
ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_join_docker_swarm_managers.yaml
# Join the worker instances to the Swarm
ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_join_docker_swarm_workers.yaml

# Build, Deploy, Test the application

# Tear down the Docker Swarm infrastructure
aws cloudformation delete-stack --region ${AWS_REGION} --stack-name ${AWS_STACK_NAME}
# Delete key pair
aws ec2 delete-key-pair --region ${AWS_REGION} --key-name ${CFN_KEYPAIR}
rm -rf ${CFN_KEYPAIR}
```
    • Commit ve push.
      
git add .
git commit -m 'added scripts for qa automation environment'
git push
git checkout dev
git merge feature/msp-16
git push origin dev

# 17. MSP 17 - Prepare a QA Automation Pipeline for Nightly Builds

QA Automation Setup for Development
Prepare a QA Automation Pipeline
MSP-17
Prepare a QA Automation Pipeline on dev branch for Nightly Builds.
feature/msp-17

    • Nightly build yapacak bir pipeline yapacagim. Neden nightly cunku gunduz calisiliyor gece kimse calismadigi icin gece boyunca build edilmis ve test edilmis halini girebiliyorsunuz. Her gece olmasi muhim degil 2-3 gecede bir de olabilir. Firmaya bagli. Gunduz ayakta olmasina gerek yok maliyet yarattigindan build test ve sonrasinda kaldiriyoruz.
    • Tumden gelip adim adim baktigimizda: Uygulamayi functional teste tabi tutmamiz lazim bunun icin calisan bir uygulamaya ihtiyacimiz var. Calismasi icin bir ortama ihtiyaci var. Ortami 5 tane ec2 ile sagliyoruz onlari da docker swarm ile orchestrate ediyoruz. Bu 5 instancei da ansible ile configure ediyorum. Bunlari da playbooklarla yapiyorum.
      
    • Feature olusturuyorum.
      
      git checkout dev
      git branch feature/msp-17
      git checkout feature/msp-17

    • Jenkins folder in altinda docker ile maven la build etmek icin package-with-maven-container.sh scriptini olusturuyoruz.
```bash
docker run --rm -v $HOME/.m2:/root/.m2 -v $WORKSPACE:/app -w /app maven:3.6-openjdk-11 mvn clean package #dockerla maveni olan bir klasor olusturuyoruz.
```
    • Her nightly build version olarak saklanir. Biz de docker image olarak saklayacagiz. Onceki pipeline imizda build lerimizde sabit bir image ismi belirlemistim ama burada imagelarimi taglemeliyim ve bu tagler her buildle birlikte farklilasmali. Bunu da otomatize hale getirmek istiyorum. ECR reposunun icine imagelar build nosu ile birlikte taglensin istiyorum. Bunun icin bir script olusturdum.
      
    • Jenkins folder klasoru altinda prepare-tags-ecr-for-dev-docker-images.sh scriptini yazdim.
```bash
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version) #proje build edildikten sonra maven workspaceindeki klasorden buildin version numarasini aliyorum ve onu degisken olarak atiyorum.
export IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-v${MVN_VERSION}-b${BUILD_NUMBER}" #env degiskeni olusturuyorum. Ecr repository ismini aliyorum app repo namei aliyorum. Admin server-v microservisin adi mvn versionla da version numarasini aliyorum. Build number da jenkinsden geliyor. Son calistirilan build numarasi geliyor.
#bu islemi butun microserviceler icin yapiyorum. Yalnica prometheus ve grafanaya sabit tag veriyorum cunku bunlari ben develop etmiyorum.
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
export IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
export IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
```
    • Image lari build etmek icin daha once bir script hazirlamistim ancak dinamik tag olusturmami saglayacak yeni bir tag hazirladim. Jenkins folder icinde build-dev-docker-images-for-ecr.sh adinda bir script hazirladim.
```bash
docker build --force-rm -t "${IMAGE_TAG_ADMIN_SERVER}" "${WORKSPACE}/spring-petclinic-admin-server" #image tag admin server I yukaridan aldim sonra jenkinsdeki workspacedeki calisacagi yeri gosterdim.
docker build --force-rm -t "${IMAGE_TAG_API_GATEWAY}" "${WORKSPACE}/spring-petclinic-api-gateway"
docker build --force-rm -t "${IMAGE_TAG_CONFIG_SERVER}" "${WORKSPACE}/spring-petclinic-config-server"
docker build --force-rm -t "${IMAGE_TAG_CUSTOMERS_SERVICE}" "${WORKSPACE}/spring-petclinic-customers-service"
docker build --force-rm -t "${IMAGE_TAG_DISCOVERY_SERVER}" "${WORKSPACE}/spring-petclinic-discovery-server"
docker build --force-rm -t "${IMAGE_TAG_HYSTRIX_DASHBOARD}" "${WORKSPACE}/spring-petclinic-hystrix-dashboard"
docker build --force-rm -t "${IMAGE_TAG_VETS_SERVICE}" "${WORKSPACE}/spring-petclinic-vets-service"
docker build --force-rm -t "${IMAGE_TAG_VISITS_SERVICE}" "${WORKSPACE}/spring-petclinic-visits-service"
docker build --force-rm -t "${IMAGE_TAG_GRAFANA_SERVICE}" "${WORKSPACE}/docker/grafana"
docker build --force-rm -t "${IMAGE_TAG_PROMETHEUS_SERVICE}" "${WORKSPACE}/docker/prometheus"
```
    • Image lari ecr a push etmek icin jenkins folderin altinda push-dev-docker-images-to-ecr.sh scriptini yazdim.
```bash
# Provide credentials for Docker to login the AWS ECR and push the images
aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY} #ECR icin erisim yetkisi aliyorum bu command ile.

docker push "${IMAGE_TAG_ADMIN_SERVER}"
docker push "${IMAGE_TAG_API_GATEWAY}"
docker push "${IMAGE_TAG_CONFIG_SERVER}"
docker push "${IMAGE_TAG_CUSTOMERS_SERVICE}"
docker push "${IMAGE_TAG_DISCOVERY_SERVER}"
docker push "${IMAGE_TAG_HYSTRIX_DASHBOARD}"
docker push "${IMAGE_TAG_VETS_SERVICE}"
docker push "${IMAGE_TAG_VISITS_SERVICE}"
docker push "${IMAGE_TAG_GRAFANA_SERVICE}"
docker push "${IMAGE_TAG_PROMETHEUS_SERVICE}"
```
    • Commit ve push ettim.
      
git add .
git commit -m 'added scripts for qa automation environment'
git push --set-upstream origin feature/msp-17

    • Bu featurda olusturdugum scriptleri test etmek istiyorum bunun icin jenkinsde freestyle bir project olusturdum.
    • Jenkins>freestyleproejct>test-msp-17-scripts ismini verdim. Branches to build kismina */feature/msp-17 yazdim. Add build step kismina execute-shell altina alttaki kodu yazdim. Yapmis oldugum script leri sirasiyle buraya koydum.
```bash
PATH="$PATH:/usr/local/bin"
APP_REPO_NAME="sezgin-repo/petclinic-app-dev" # Write your own repo name
AWS_REGION="us-east-1" 
ECR_REGISTRY="046402772087.dkr.ecr.us-east-1.amazonaws.com" # Replace this line with your ECR name
aws ecr create-repository \
    --repository-name ${APP_REPO_NAME} \
    --image-scanning-configuration scanOnPush=false \
    --image-tag-mutability MUTABLE \
    --region ${AWS_REGION}
. ./jenkins/package-with-maven-container.sh #maven ile build ettim.
. ./jenkins/prepare-tags-ecr-for-dev-docker-images.sh #tagleri hazirladim.
. ./jenkins/build-dev-docker-images-for-ecr.sh #imagelari build ettim
. ./jenkins/push-dev-docker-images-to-ecr.sh #imagelari push ettim
```
    • build ettim. Scriptlerimde sorun olmadigini gordum.
    • Daha once localde build etmek icin olusturmus oldugum bir docker-compose file im vardi ancak bu docker-compose file indaki image isimleri statik ama ben artik her buildde her push da tagleri degisen imagelar olusturuyorum. Benim de bu dinamik bir sekilde olusturdum imagelarin taglerini ecr dan alabilmem lazim. Bunun icin docker-composeumu da buna uygun hale getirmem gerekiyor.
    • Petclinic in altinda docker-compose-swarm-dev.yml adinda bir docker-compose file yazdim.
```yaml
Version: '3.8' #version degisti ama cok onemli degil

services:
  config-server:
    image: "${IMAGE_TAG_CONFIG_SERVER}" #en buyuk fark tag degisimi
    networks:
      - clarusnet
    ports:
     - 8888:8888

  discovery-server:
    image: "${IMAGE_TAG_DISCOVERY_SERVER}"
    depends_on:
      - config-server
    entrypoint: ["./dockerize","-wait=tcp://config-server:8888","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
     - 8761:8761

  customers-service:
    image: "${IMAGE_TAG_CUSTOMERS_SERVICE}"
    deploy:
      replicas: 3 #replicalari duzenledim. Cunku uygulamamin her zaman calistigina emin olmak istedim.
      update_config:
          parallelism: 2
          delay: 5s
          order: start-first
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
    - 8081:8081

  visits-service:
    image: "${IMAGE_TAG_VISITS_SERVICE}"
    deploy:
      replicas: 3
      update_config:
          parallelism: 2
          delay: 5s
          order: start-first
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
     - 8082:8082

  vets-service:
    image: "${IMAGE_TAG_VETS_SERVICE}"
    deploy:
      replicas: 3
      update_config:
          parallelism: 2
          delay: 5s
          order: start-first
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
     - 8083:8083

  api-gateway:
    image: "${IMAGE_TAG_API_GATEWAY}"
    deploy:
      replicas: 3
      update_config:
          parallelism: 2
          delay: 5s
          order: start-first
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
     - 8080:8080

  tracing-server:
    image: openzipkin/zipkin
    environment:
    - JAVA_OPTS=-XX:+UnlockExperimentalVMOptions -Djava.security.egd=file:/dev/./urandom
    networks:
      - clarusnet
    ports:
     - 9411:9411

  admin-server:
    image: "${IMAGE_TAG_ADMIN_SERVER}"
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
     - 9090:9090

  hystrix-dashboard:
    image: "${IMAGE_TAG_HYSTRIX_DASHBOARD}"
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
     - 7979:7979

  ## Grafana / Prometheus

  grafana-server:
    image: "${IMAGE_TAG_GRAFANA_SERVICE}"
    networks:
      - clarusnet
    ports:
    - 3000:3000

  prometheus-server:
    image: "${IMAGE_TAG_PROMETHEUS_SERVICE}"
    networks:
      - clarusnet
    ports:
    - 9091:9090

networks:
  clarusnet:
    driver: overlay

    • ansible/scripts klasorunun altinda pb_deploy_app_on_docker_swarm.yaml adinda bir playbook olusturdum.

---
- hosts: role_grand_master #grand mastera gidiyor
  tasks:
  - name: Copy docker compose file to grand master
    copy:
      src: "{{ workspace }}/docker-compose-swarm-dev-tagged.yml" #icerisine docker-compose-swarm-dev-tagged.yml dosyasini atiyor ama bunu envsubt komutu ile olusturdugumuz dosyadaki degerleri atiyor. Bu komutun aciklamasi asagida var. 
      dest: /home/ec2-user/docker-compose-swarm-dev-tagged.yml #dosyayi grand-master icindeki kok dizine atiyor. Kendi workspace imizde goremeyecegiz. Icerisinde Docker-compose olan ec2 ya atiyoruz. Cunku docker-compose buildi orada yapacagiz. 

  - name: get login credentials for ecr
    shell: "export PATH=$PATH:/usr/local/bin/ && aws ecr get-login-password --region {{ aws_region }} | docker login --username AWS --password-stdin {{ ecr_registry }}" #ecr a baglanip oradaki imagelari cekmek icin 

  - name: deploy the app stack on swarm
    shell: "docker stack deploy --with-registry-auth -c /home/ec2-user/docker-compose-swarm-dev-tagged.yml {{ app_name }}"
    register: output

- debug: msg="{{ output.stdout }}"
```
    • ansible/playbooks klasorunun altinda deploy_app_on_docker_swarm.sh adinda bir script olusturdum.
```bash
PATH="$PATH:/usr/local/bin" #ansible in calismasi gerekn path
APP_NAME="petclinic" #app name adi
envsubst < docker-compose-swarm-dev.yml > docker-compose-swarm-dev-tagged.yml #docker-compose-swarm-dev.yml icindeki degiskenleri aliyor mevcut o andaki degiskenle ile degistirip bize statik bir docker swam dosyasi olusturuyor. docker-compose-swarm-dev-tagged.yml bu dosyanin icine value larini yaziyor. Yani bu dosya icinde olusturulan tum env variable lari alma imkanim oluyor. Her pipeline I build ettigimde oradaki degiskenleri ayri bir dosyaya yazdirmis oluyorum. 
ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b --extra-vars "workspace=${WORKSPACE} app_name=${APP_NAME} aws_region=${AWS_REGION} ecr_registry=${ECR_REGISTRY}" ./ansible/playbooks/pb_deploy_app_on_docker_swarm.yaml #playbook olustururken degiskenlerimin hepsini belirtmek zorundayim. Dinamik olarak olusturdugum ansible dosyasi icerisindeki tagleri buraya yazmam gerekiyor. 
```
    • Selenium calismasi icin chrome un calismasi lazim. Ama instance larda chrome yok. Bu komutla arka planda chromeu gormeden testlerim yapiliyor. Pipelineimi faal hale getirmeden once functional testlerim yapilacagi ortami da test etmek istiyorum. Bunun icin Selenium-jobs folderin icine dummy_selenium_test_headless.py fileini yaziyorum.
```js
from selenium import webdriver

chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("headless")
chrome_options.add_argument("no-sandbox")
chrome_options.add_argument("disable-dev-shm-usage")
driver = webdriver.Chrome(options=chrome_options)

base_url = "https://www.google.com/"
driver.get(base_url)
source = driver.page_source

if "I'm Feeling Lucky" in source:
  print("Test passed")
else:
  print("Test failed")
driver.close()
```
    • ansible/playbooks un icine pb_run_dummy_selenium_job.yaml filei ni yerlestirdim.
```yaml      
---
- hosts: all
  tasks:
  - name: run dummy selenium job
    shell: "docker run --rm -v {{ workspace }}:{{ workspace }} -w {{ workspace }} callahanclarus/selenium-py-chrome:latest python {{ item }}" #bu image ile bu testi yapiyorum. Bu image in icinde chrome web driver ve selenium yuklu. Dolayisiyla ben functional testimi yapabilirim
    with_fileglob: "{{ workspace }}/selenium-jobs/dummy*.py" #ansible loopla ayni gorevi yapiyor. Bu workspace giriyor buradaki dummy ile baslayan tum dosyalari aliyor ve yukarida python {{ item }} yazdigimiz icin o item in icine bu komutu yazdigi icin bunu calistiriyor. Su an selenium testlerini yaptigim icin bir kere calistiricak
    register: output 
  
  - name: show results #sonuclari da ekrana yazdir komutu
    debug: msg="{{ item.stdout }}"
    with_items: "{{ output.results }}"
```
    • Bu olusturdugumuz ortami calistirmak icin Ansible/scripts klasorunun altina run_dummy_selenium_job.sh ekliyorum.
```bash
PATH="$PATH:/usr/local/bin"
ansible-playbook --connection=local --inventory 127.0.0.1, --extra-vars "workspace=${WORKSPACE}" ./ansible/playbooks/pb_run_dummy_selenium_job.yaml
```
    • Commit ve push ettim.
      
git add .
git commit -m 'added scripts for running dummy selenium job'
git push --set-upstream origin feature/msp-17

    • Son olusturugum scripti denemek icin bir freestyle project olusturdum. Jenkins>Dashboard>newitem>freestyleproject>test-running-dummy-selenium-job OK dedim. Source code management>git> repository url yapistirdim. Branches to build>feature/msp-17. Add build step kismina son scriptimi yazdim.
```bash
PATH="$PATH:/usr/local/bin"
ansible-playbook --connection=local --inventory 127.0.0.1, --extra-vars "workspace=${WORKSPACE}" ./ansible/playbooks/pb_run_dummy_selenium_job.yaml
```
    • Build ettim. Bu job benim jenkins serverimda olustu ama normalde ben bunun benim olusturacagim ec2 larin icinde olusmasini istiyorum. Bunun icin env hazirlamam gerekiyor.

    • Gercek fonksiyonal testlerimi olusturmak icin playbookumu olusturuyorum. selenium-jobs folderinin altina pb_run_selenium_jobs.yaml dosyasini ekliyorum. 
```yaml
---
- hosts: all #dummy_run yaml dan tek farki burada py ile test* ile baslayan tum test dosyalarini test ediyoruz. Ayni image ile test ediyoruz. Ayni ciktilari istiyorum. Yine ayni sekilde loop yapiyorum. Dummy test yalnizca test amacli imagein calisip calismadigini test amacli bir yaml di bu ise tum testlerin calistirilmasi amaclanan bir yaml.
  tasks:
  - name: run all selenium jobs
    shell: "docker run --rm --env MASTER_PUBLIC_IP={{ master_public_ip }} -v {{ workspace }}:{{ workspace }} -w {{ workspace }} callahanclarus/selenium-py-chrome:latest python {{ item }}"
    register: output
    with_fileglob: "{{ workspace }}/selenium-jobs/test*.py"
  
  - name: show results
    debug: msg="{{ item.stdout }}"
    with_items: "{{ output.results }}"
```
    • Yukaridaki playbooku calistirmak icin script yazdim. ansible/scripts altina run_selenium_jobs.sh sciptini yazdim.
```bash
PATH="$PATH:/usr/local/bin"
ansible-playbook -vvv --connection=local --inventory 127.0.0.1, --extra-vars "workspace=${WORKSPACE} master_public_ip=${GRAND_MASTER_PUBLIC_IP}" ./ansible/playbooks/pb_run_selenium_jobs.yaml #-vvv(detayli log almak icin kullandim). 
```
    • Nightliy pipeline icin jenkins dosyasinin altinda jenkinsfile-petclinic-nightly bir file olusturdum.
```groovy
pipeline {
    agent { label "master" }
    environment {
        PATH=sh(script:"echo $PATH:/usr/local/bin", returnStdout:true).trim() //komutu shell script olarak calistir ve bunu returnStdout olarak PATH degiskenine ata. Aws cli in nerede calisacagini gosterdim.
        APP_NAME="petclinic"
        APP_STACK_NAME="sezgin-${APP_NAME}-app-${BUILD_NUMBER}" //appname yukardan, buildNumber jenkinsden gelecek
        APP_REPO_NAME="sezgin-repo/${APP_NAME}-app-dev"
        AWS_ACCOUNT_ID=sh(script:'export PATH="$PATH:/usr/local/bin" && aws sts get-caller-identity --query Account --output text', returnStdout:true).trim() //shelldeki bu komutu aws_account_id icine ata
        AWS_REGION="us-east-1"
        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        CFN_KEYPAIR="sezgin-${APP_NAME}-dev-${BUILD_NUMBER}.key"
        CFN_TEMPLATE="./infrastructure/dev-docker-swarm-infrastructure-cfn-template.yml" //5 tane bos ec2 olusturmak icin cfn template ismi
        ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${CFN_KEYPAIR}" #ansible ec2 lara ulasmak icin keypairi buradan aliyor.
        ANSIBLE_HOST_KEY_CHECKING="False" //ansible baglanirken yes sorusunun sormasini engellemek icin
    }
//ecr repomu olusturuyorum. 
    stages {
        stage('Create ECR Repo') {
            steps {
                echo "Creating ECR Repo for ${APP_NAME} app"
                sh """
                aws ecr create-repository \
                    --repository-name ${APP_REPO_NAME} \
                    --image-scanning-configuration scanOnPush=false \
                    --image-tag-mutability MUTABLE \
                    --region ${AWS_REGION}
                """
            }
        }
//maven ile app I packaging yaptim bunlari grovy diline cevirerek yazdim
        stage('Package Application') {
            steps {
                echo 'Packaging the app into jars with maven'
                sh ". ./jenkins/package-with-maven-container.sh"
            }
        }
        stage('Prepare Tags for Docker Images') {
            steps {
                echo 'Preparing Tags for Docker Images'
                script {
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim() //trim komutun basindaki ve sonundaki bosluklari atiyor
                    env.IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    env.IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
                    env.IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
                }
            }
        }
        stage('Build App Docker Images') {
            steps {
                echo 'Building App Dev Images'
                sh ". ./jenkins/build-dev-docker-images-for-ecr.sh" //imagelari build ettigim komut
                sh 'docker image ls'
            }
        }
//imagelari ecra push ediyorum
        stage('Push Images to ECR Repo') {
            steps {
                echo "Pushing ${APP_NAME} App Images to ECR Repo"
                sh ". ./jenkins/push-dev-docker-images-to-ecr.sh"
            }
        }
//Ansible icin keypair olusturup bunu yazdiriyor ve okuma yetkisi verdim
        stage('Create Key Pair for Ansible') {
            steps {
                echo "Creating Key Pair for ${APP_NAME} App"
                sh "aws ec2 create-key-pair --region ${AWS_REGION} --key-name ${CFN_KEYPAIR} --query KeyMaterial --output text > ${CFN_KEYPAIR}"
                sh "chmod 400 ${CFN_KEYPAIR}"
            }
        }
//cf ile 5 ec2 olusturdum
        stage('Create QA Automation Infrastructure') {
            steps {
                echo 'Creating QA Automation Infrastructure for Dev Environment with Cloudfomation'
                sh "aws cloudformation create-stack --region ${AWS_REGION} --stack-name ${APP_STACK_NAME} --capabilities CAPABILITY_IAM --template-body file://${CFN_TEMPLATE} --parameters ParameterKey=KeyPairName,ParameterValue=${CFN_KEYPAIR}"
//Grandmaster ayaga kalkana kadar bu dongu devam ediyor. GrandMasterin public IP sinin atanip atanmadigina bakarak kontrol ettim
                script {
                    while(true) {
                        echo "Docker Grand Master is not UP and running yet. Will try to reach again after 10 seconds..."
                        sleep(10)

                        ip = sh(script:"aws ec2 describe-instances --region ${AWS_REGION} --filters Name=tag-value,Values=grand-master Name=tag-value,Values=${APP_STACK_NAME} --query Reservations[*].Instances[*].[PublicIpAddress] --output text", returnStdout:true).trim()

                        if (ip.length() >= 7) {
                            echo "Docker Grand Master Public Ip Address Found: $ip"
                            env.GRAND_MASTER_PUBLIC_IP = "$ip"
                            break
                        }
                    }
//ssh baglantisi icin de bir dongu yazdim
                    while(true) {
                        try{
                            sh "ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -i ${WORKSPACE}/${CFN_KEYPAIR} ec2-user@${GRAND_MASTER_PUBLIC_IP} hostname"
                            echo "Docker Grand Master is reachable with SSH."
                            break
                        }
                        catch(Exception){
                            echo "Could not connect to Docker Grand Master with SSH, I will try again in 10 seconds"
                            sleep(10)
                        }
                    }
                }
            }
        }
//Ansible ile olusturdum playbooklari sirasiyla calistiriyorum
        stage('Create Docker Swarm for QA Automation Build') {
            steps {
                echo "Setup Docker Swarm for QA Automation Build for ${APP_NAME} App"
                echo "Update dynamic environment"
                sh "sed -i 's/APP_STACK_NAME/${APP_STACK_NAME}/' ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml"
                echo "Swarm Setup for all nodes (instances)"
                sh "ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_setup_for_all_docker_swarm_instances.yaml"
                echo "Swarm Setup for Grand Master node"
                sh "ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_initialize_docker_swarm.yaml"
                echo "Swarm Setup for Other Managers nodes"
                sh "ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_join_docker_swarm_managers.yaml"
                echo "Swarm Setup for Workers nodes"
                sh "ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_join_docker_swarm_workers.yaml"
            }
        }
//Uygulamami docker swarm clusterina deploy ettim
        stage('Deploy App on Docker Swarm'){
            steps {
                echo 'Deploying App on Swarm'
                sh 'envsubst < docker-compose-swarm-dev.yml > docker-compose-swarm-dev-tagged.yml'
                sh 'ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml -b --extra-vars "workspace=${WORKSPACE} app_name=${APP_NAME} aws_region=${AWS_REGION} ecr_registry=${ECR_REGISTRY}" ./ansible/playbooks/pb_deploy_app_on_docker_swarm.yaml'
            }
        }
//uygulamanin calisip calismadigini kontrol ediyorum. Eger kontrol etmezsem teste gecebilir oyle olunca hatali sonuc alabilirim. Bunun icin bir dongu yazdim ve calistigindan emin olduktan sonra teste gectim
        stage('Test the Application Deployment'){
            steps {
                echo "Check if the ${APP_NAME} app is ready or not"
                script {

                    while(true) {
                        try{
                            sh "curl -s ${GRAND_MASTER_PUBLIC_IP}:8080"
                            echo "${APP_NAME} app is successfully deployed."
                            break
                        }
                        catch(Exception){
                            echo "Could not connect to ${APP_NAME} app"
                            sleep(5)
                        }
                    }
                }
            }
        }
//Selenium Jobs larin calistirilmasi asamasi
        stage('Run QA Automation Tests'){
            steps {
                echo "Run the Selenium Functional Test on QA Environment"
                sh 'ansible-playbook -vvv --connection=local --inventory 127.0.0.1, --extra-vars "workspace=${WORKSPACE} master_public_ip=${GRAND_MASTER_PUBLIC_IP}" ./ansible/playbooks/pb_run_selenium_jobs.yaml'
            }
        }
    }
//pipelinein sonuda ne olacagini gosteriyorum. Always yani basarili olursa tum infrastructure silinecek. Faille de ilgili bir sart yazilabilirdi.
    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
            echo 'Delete the Image Repository on ECR'
            sh """
                aws ecr delete-repository \
                  --repository-name ${APP_REPO_NAME} \
                  --region ${AWS_REGION}\
                  --force
                """
            echo 'Tear down the Docker Swarm infrastructure using AWS CLI'
            sh "aws cloudformation delete-stack --region ${AWS_REGION} --stack-name ${APP_STACK_NAME}"
            echo "Delete existing key pair using AWS CLI"
            sh "aws ec2 delete-key-pair --region ${AWS_REGION} --key-name ${CFN_KEYPAIR}"
            sh "rm -rf ${CFN_KEYPAIR}"
        }
    }
}
```
    • Commit ve push

git add .
git commit -m 'added qa automation pipeline for dev'
git push
git checkout dev
git merge feature/msp-17
git push origin dev

    • jenkins>dashboard>newitem>pipeline>petclinic nightly OK. Build Triggers>Build perdiocally(Cron Job)>(0 0 * * *) olarak ayarladim. Source Code Management>git>Girhub url. Branch>dev olacak. Script Path>Jenkins>jenkinsfile-petclinic-nightly SAVE.
    • Build ettim.
    • Basarili oldu. CF silindi tum instancelar silindi.
      
# 18. MSP 18 - Create a QA Environment on Docker Swarm with Cloudformation and Ansible

QA Setup for Release
Create a Key Pair for QA Environment
MSP-18-1
Create a permanent Key Pair for Ansible to be used in QA Environment on Release.
feature/msp-18
QA Setup for Release
Create a QA Infrastructure with AWS Cloudformation
MSP-18-2
Create a Permanent QA Infrastructure for Docker Swarm with AWS Cloudformation.
feature/msp-18
QA Setup for Release
Create a QA Environment with Docker Swarm
MSP-18-3
Create a QA Environment for Release with Docker Swarm.
feature/msp-18

    • QA testlerinin yapilmasi icin 3. pipelinei kurdum. Release branchinda bir pipeline olusturacagim.

    • Feature olusturdum.

git checkout dev
git branch feature/msp-18
git checkout feature/msp-18

    • Onceki pipelinedaki infrastructureun simdi olusturacagimdan bir farki yok. Ayni environmenti kullaniyorum. Tek farki environment tag lerim dev olarak taglemistim ama bunda ise tek fark taginin qa olmasi ayni da  olabilirdi ama buyuk bir sirkette olsa takimlar farkli olurdu o zaman gormek acisindan boyle kullanmak daha iyi.
      
    • Infrastructure>docker-swarm-infrastructure-cfn-template.yml olusturdum.
```yaml
AWSTemplateFormatVersion: 2010-09-09

Description: >
  This Cloudformation Template creates an infrastructure for Docker Swarm
  with five EC2 Instances with Amazon Linux 2. Instances are configured
  with custom security group allowing SSH (22), HTTP (80) UDP (4789, 7946), 
  and TCP(2377, 7946, 8080) connections from anywhere.
  User needs to select appropriate key name when launching the template.
Parameters:
  KeyPairName:
    Description: Enter the name of your Key Pair for SSH connections.
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: Must one of the existing EC2 KeyPair

Resources:  
  RoleEnablingEC2forECR:
    Type: "AWS::IAM::Role"
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              Service:
              - ec2.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess
  EC2Profile:
    Type: "AWS::IAM::InstanceProfile"
    Properties:
      Roles: #required
        - !Ref RoleEnablingEC2forECR
  DockerMachinesSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH and HTTP for Docker Machines
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 2377
          ToPort: 2377
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 7946
          ToPort: 7946
          CidrIp: 0.0.0.0/0
        - IpProtocol: udp
          FromPort: 7946
          ToPort: 7946
          CidrIp: 0.0.0.0/0
        - IpProtocol: udp
          FromPort: 4789
          ToPort: 4789
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 8080
          ToPort: 8080
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 8088
          ToPort: 8088
          CidrIp: 0.0.0.0/0
  DockerMachineLT:
    Type: "AWS::EC2::LaunchTemplate"
    Properties:
      LaunchTemplateData:
        ImageId: ami-0947d2ba12ee1ff75
        InstanceType: t2.medium
        KeyName: !Ref KeyPairName
        IamInstanceProfile: 
          Arn: !GetAtt EC2Profile.Arn
        SecurityGroupIds:
          - !GetAtt DockerMachinesSecurityGroup.GroupId
        TagSpecifications: 
          - ResourceType: instance
            Tags: 
              - Key: app-stack-name
                Value: !Sub ${AWS::StackName}
              - Key: environment
                Value: qa
  DockerInstance1:
    Type: AWS::EC2::Instance
    DependsOn:
        - "DockerInstance2"
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref DockerMachineLT
        Version: !GetAtt DockerMachineLT.LatestVersionNumber
      Tags:                
        - Key: server
          Value: docker-instance-1                       
        - Key: swarm-role
          Value: grand-master                       
        - Key: Name
          Value: !Sub ${AWS::StackName} Docker Machine 1st        
  DockerInstance2:
    Type: AWS::EC2::Instance
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref DockerMachineLT
        Version: !GetAtt DockerMachineLT.LatestVersionNumber
      Tags:                
        - Key: server
          Value: docker-instance-2                       
        - Key: swarm-role
          Value: manager                       
        - Key: Name
          Value: !Sub ${AWS::StackName} Docker Machine 2nd
  DockerInstance3:
    Type: AWS::EC2::Instance
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref DockerMachineLT
        Version: !GetAtt DockerMachineLT.LatestVersionNumber
      Tags:                
        - Key: server
          Value: docker-instance-3                       
        - Key: swarm-role
          Value: manager                       
        - Key: Name
          Value: !Sub ${AWS::StackName} Docker Machine 3rd
  DockerInstance4:
    Type: AWS::EC2::Instance
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref DockerMachineLT
        Version: !GetAtt DockerMachineLT.LatestVersionNumber
      Tags:                
        - Key: server
          Value: docker-instance-4                       
        - Key: swarm-role
          Value: worker                       
        - Key: Name
          Value: !Sub ${AWS::StackName} Docker Machine 4th
  DockerInstance5:
    Type: AWS::EC2::Instance
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref DockerMachineLT
        Version: !GetAtt DockerMachineLT.LatestVersionNumber
      Tags:                
        - Key: server
          Value: docker-instance-5                       
        - Key: swarm-role
          Value: worker                       
        - Key: Name
          Value: !Sub ${AWS::StackName} Docker Machine 5th
Outputs:
  1stDockerInstanceDNSName:
    Description: 1st Docker Instance DNS Name
    Value: !Sub 
      - ${PublicAddress}
      - PublicAddress: !GetAtt DockerInstance1.PublicDnsName
  2ndDockerInstanceDNSName:
    Description: 2nd Docker Instance DNS Name
    Value: !Sub 
      - ${PublicAddress}
      - PublicAddress: !GetAtt DockerInstance2.PublicDnsName
  3rdDockerInstanceDNSName:
    Description: 3rd Docker Instance DNS Name
    Value: !Sub 
      - ${PublicAddress}
      - PublicAddress: !GetAtt DockerInstance3.PublicDnsName
  4thDockerInstanceDNSName:
    Description: 4th Docker Instance DNS Name
    Value: !Sub 
      - ${PublicAddress}
      - PublicAddress: !GetAtt DockerInstance4.PublicDnsName
  5thDockerInstanceDNSName:
    Description: 5th Docker Instance DNS Name
    Value: !Sub 
      - ${PublicAddress}
      - PublicAddress: !GetAtt DockerInstance5.PublicDnsName
```
    • Keypair olusturmak icin Jenkins>create-permanent-key-pair-for-qa-environment.sh scriptini olusturdum.
```bash
PATH="$PATH:/usr/local/bin"
APP_NAME="petclinic"
CFN_KEYPAIR="sezgin-${APP_NAME}-qa.key"
AWS_REGION="us-east-1"
aws ec2 create-key-pair --region ${AWS_REGION} --key-name ${CFN_KEYPAIR} --query "KeyMaterial" --output text > ${CFN_KEYPAIR}
chmod 400 ${CFN_KEYPAIR}
mkdir -p ${JENKINS_HOME}/.ssh
mv ${CFN_KEYPAIR} ${JENKINS_HOME}/.ssh/${CFN_KEYPAIR}
ls -al ${JENKINS_HOME}/.ssh
```
    • infrastructurei calistirmak icin infrastructure>create-qa-infrastructure-cfn.sh scriptini yazdim.
```bash
PATH="$PATH:/usr/local/bin"
APP_NAME="petclinic"
APP_STACK_NAME="sezgin-$APP_NAME-App-QA-${BUILD_NUMBER}"
CFN_KEYPAIR="sezgin-${APP_NAME}-qa.key"
CFN_TEMPLATE="./infrastructure/qa-docker-swarm-infrastructure-cfn-template.yml"
AWS_REGION="us-east-1"
aws cloudformation create-stack --region ${AWS_REGION} --stack-name ${APP_STACK_NAME} --capabilities CAPABILITY_IAM --template-body file://${CFN_TEMPLATE} --parameters ParameterKey=KeyPairName,ParameterValue=${CFN_KEYPAIR}
```
    • 5 tane instancei bu scriptle olusturacagim. Simdi ansible ile confugurasyonlarini yapmam lazim. Daha once dev_stack_dynamic_inventory_aws_ec2.yaml dosyasi ile bunu yapmistim. Simdi ise ansible/inventory>qa_stack_dynamic_inventory_aws_ec2.yaml dosyasini olusturuyorum. Bunlar arasindaki fark taglemeler farkli orada dev ile taglemistim simdi qa ile tagledim.
```yaml
plugin: aws_ec2
regions:
  - "us-east-1"
filters:
  tag:app-stack-name: APP_STACK_NAME
  tag:environment: qa
keyed_groups:
  - key: tags['app-stack-name']
    prefix: 'app_stack_'
    separator: ''
  - key: tags['swarm-role']
    prefix: 'role_'
    separator: ''
  - key: tags['environment']
    prefix: 'env_'
    separator: ''
  - key: tags['server']
    separator: ''
hostnames:
  - "private-ip-address"
compose:
  ansible_user: "'ec2-user'"
```
    • Ansible configurasyonunu calistirmak icin ansible/scripts>qa_build_deploy_environment.sh scriptini yazdim. Bir onceki pipeline da yazdigim playbooklarin aynisini kullanacagim. Bu script de aynisi olacak.
```bash
PATH="$PATH:/usr/local/bin"
APP_NAME="petclinic"
CFN_KEYPAIR="sezgin-${APP_NAME}-qa.key"
APP_STACK_NAME="sezgin-$APP_NAME-App-QA-${BUILD_NUMBER}"
export ANSIBLE_PRIVATE_KEY_FILE="${JENKINS_HOME}/.ssh/${CFN_KEYPAIR}"
export ANSIBLE_HOST_KEY_CHECKING=False
sed -i "s/APP_STACK_NAME/$APP_STACK_NAME/" ./ansible/inventory/qa_stack_dynamic_inventory_aws_ec2.yaml
# Swarm Setup for all nodes (instances)
ansible-playbook -i ./ansible/inventory/qa_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_setup_for_all_docker_swarm_instances.yaml
# Swarm Setup for Grand Master node
ansible-playbook -i ./ansible/inventory/qa_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_initialize_docker_swarm.yaml
# Swarm Setup for Other Managers nodes
ansible-playbook -i ./ansible/inventory/qa_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_join_docker_swarm_managers.yaml
# Swarm Setup for Workers nodes
ansible-playbook -i ./ansible/inventory/qa_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_join_docker_swarm_workers.yaml
```

    • Weekly pipleline icin 2 tane jenkins file gerekli 1.si environment in yani ec2 larin bulundugu infrastructurein olusturulmasi 2.si ise uygulamanin deploy edilmesi. Benim infrastructureim her zaman ayakta kalacak ben uygulamayi yeniden deploy edecegim bu nedenle infrastructre in bozulmasini istemiyorum. Uygulamayi deploy etmek icin 2. jenkinsfile i yazdim. Jenkins>jenkinsfile-create-qa-environment-on-docker-swarm adinda bir jenkinsfile olusturdum.
```groovy
pipeline {
    agent { label "master" }
    environment {
        PATH=sh(script:"echo $PATH:/usr/local/bin", returnStdout:true).trim()
        APP_NAME="petclinic"
        APP_STACK_NAME="sezgin-$APP_NAME-App-QA-${BUILD_NUMBER}"
        AWS_REGION="us-east-1"
        CFN_KEYPAIR="sezgin-${APP_NAME}-qa.key"
        CFN_TEMPLATE="./infrastructure/qa-docker-swarm-infrastructure-cfn-template.yml"
        ANSIBLE_PRIVATE_KEY_FILE="${JENKINS_HOME}/.ssh/${CFN_KEYPAIR}"
        ANSIBLE_HOST_KEY_CHECKING="False"
    }
    stages {
        stage('Create QA Environment Infrastructure') {
            steps {
                echo 'Creating Infrastructure for QA Environment with Cloudfomation'
                sh "aws cloudformation create-stack --region ${AWS_REGION} --stack-name ${APP_STACK_NAME} --capabilities CAPABILITY_IAM --template-body file://${CFN_TEMPLATE} --parameters ParameterKey=KeyPairName,ParameterValue=${CFN_KEYPAIR}"
//ip adresini alarak instancein ayakta olup olmadigini kontrol eden bir dongu yazdim
                script {
                    while(true) {
                        echo "Docker Grand Master is not UP and running yet. Will try to reach again after 10 seconds..."
                        sleep(10)

                        ip = sh(script:"aws ec2 describe-instances --region ${AWS_REGION} --filters Name=tag-value,Values=grand-master Name=tag-value,Values=${APP_STACK_NAME} --query Reservations[*].Instances[*].[PublicIpAddress] --output text", returnStdout:true).trim()

                        if (ip.length() >= 7) {
                            echo "Docker Grand Master Public Ip Address Found: $ip"
                            env.GRAND_MASTER_PUBLIC_IP = "$ip"
                            break
                        }
                    }
//ssh ile baglanana kadar beklemesi icin bir dongu yazdim.Baglanti saglandi ise diger 4 u de hazidir onun icin bu sekilde yaptim
                    while(true) {
                        try{
                            sh "ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -i ${JENKINS_HOME}/.ssh/${CFN_KEYPAIR} ec2-user@${GRAND_MASTER_PUBLIC_IP} hostname"
                            echo "Docker Grand Master is reachable with SSH."
                            break
                        }
                        catch(Exception){
                            echo "Could not connect to Docker Grand Master with SSH, I will try again in 10 seconds"
                            sleep(10)
                        }
                    }
                }
            }
        }
//Ilk asama infrastructure kurmak 2.asama docker swarmi aktif hale getirmek. Bunun icin daha once kullandigim playbooklarin aynisini kullaniyorum. Grandmastera docker, aws cli, docker compose kurup sonra managerlari grand mastera, workerlari da managerlara baglayarak 5 li bir orchestration kurarak instancelarimi faal hale getiriyorum.
        stage('Create Docker Swarm for QA Environment') {
            steps {
                echo "Setup Docker Swarm for QA Environment for ${APP_NAME} App"
                echo "Update dynamic environment"
                sh "sed -i 's/APP_STACK_NAME/${APP_STACK_NAME}/' ./ansible/inventory/qa_stack_dynamic_inventory_aws_ec2.yaml"
                echo "Swarm Setup for all nodes (instances)"
                sh "ansible-playbook -i ./ansible/inventory/qa_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_setup_for_all_docker_swarm_instances.yaml"
                echo "Swarm Setup for Grand Master node"
                sh "ansible-playbook -i ./ansible/inventory/qa_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_initialize_docker_swarm.yaml"
                echo "Swarm Setup for Other Managers nodes"
                sh "ansible-playbook -i ./ansible/inventory/qa_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_join_docker_swarm_managers.yaml"
                echo "Swarm Setup for Workers nodes"
                sh "ansible-playbook -i ./ansible/inventory/qa_stack_dynamic_inventory_aws_ec2.yaml -b ./ansible/playbooks/pb_join_docker_swarm_workers.yaml"
            }
        }
    }
//failure olursa bu pipelinei sil oncekinde basarili olursa siliniyordu bunun farki ise bunun ayakta kalmasini istiyoruz cunku bu environmentda manuel testerlar calisacak.
    post {
        failure {
            echo 'Tear down the Docker Swarm infrastructure using AWS CLI'
            sh "aws cloudformation delete-stack --region ${AWS_REGION} --stack-name ${APP_STACK_NAME}"
        }
    }
}
```
    • Commit ve push.

git add .
git commit -m "added jenkinsfile for creating manual qa environment"
git push --set-upstream origin feature/msp-18
git checkout dev
git merge feature/msp-18
git push origin dev

    • Buradaki key sabit bir key olacak cunku 1 hafta ayakta kalacak. Weekly job icin Jenkins>Dashboard>new item>freestyle>create-permanent-key-pair-for-petclinic-qa-env isimli bir jenkins job OK. Build>execute shell
```bash
PATH="$PATH:/usr/local/bin"
APP_NAME="petclinic"
CFN_KEYPAIR="sezgin-${APP_NAME}-qa.key"
AWS_REGION="us-east-1"
aws ec2 create-key-pair --region ${AWS_REGION} --key-name ${CFN_KEYPAIR} --query "KeyMaterial" --output text > ${CFN_KEYPAIR}
chmod 400 ${CFN_KEYPAIR}
mkdir -p ${JENKINS_HOME}/.ssh //key in surekli kalmasi icin jenkins home a tasiyorum. Onun icin once ssh dosyasi actim
mv ${CFN_KEYPAIR} ${JENKINS_HOME}/.ssh/${CFN_KEYPAIR}
ls -al ${JENKINS_HOME}/.ssh //key in surekli kalmasi icin jenkins home a tasiyorum.
```
    • BUILD. Key olustu jenkins instanceimin icinde gorebiliyorum.
      
    • Jenkins>Dasghboard>pipeline>create-qa-environment-on-docker-swarm OK. Git source>project URL yazdim. Branches to build>*/dev yazdim. Script path>jenkins>jenkinsfile-create-qa-environment-on-docker-swarm SAVE.
    • BUILD. 5 instance in oldugu environment hazir durumda. Bundan sonraki asama deploy asamasi olacak bundan sonra bununla ilgili hazirliklari yapacagim.
      
# 19. MSP 19 - Prepare Build Scripts for QA Environment

QA Setup for Release
Prepare Build Scripts for QA Environment
MSP-19
Prepare Build Scripts for creating ECR Repo for QA, building QA Docker images, deploying app with Docker Compose.
feature/msp-19

    • Yeni feature olusturdum.
      
git checkout dev
git branch feature/msp-19
git checkout feature/msp-19

    • Su asamada imagelari push edecek bir repom yok. Repoyu olusturacak bir script olusturmam lazim ve bu imagelari tagleyecek bir script yazmam lazim. 
    • Ecr repo olustruyorum. jenkins>dashboard>newItem>freeStyle>create-ecr-docker-registry-for-petclinic-qa OK. ExecuteShell alttaki komutu yazdim.
```bash
PATH="$PATH:/usr/local/bin"
APP_REPO_NAME="sezgin-repo/petclinic-app-qa" //farkli olarak yalnizca dev yerine qa yazdim
AWS_REGION="us-east-1"

aws ecr create-repository \
  --repository-name ${APP_REPO_NAME} \
  --image-scanning-configuration scanOnPush=false \
  --image-tag-mutability MUTABLE \
  --region ${AWS_REGION}
```
    • BUILD. Repom olustu.
    • Image lari push edecegim repository hazir once imagelarimi tagliyorum. Daha once bu komutu acikladim, buradaki tek farki dev degil qa olarak taglenmis durumda. jenkins>prepare-tags-ecr-for-qa-docker-images.sh icine scriptimi yaziyorum.
```bash
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
export IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
export IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
```
    • imagelarimi build etmek icin jenkins>build-qa-docker-images-for-ecr.sh scripti yazdim. 
```bash
docker build --force-rm -t "${IMAGE_TAG_ADMIN_SERVER}" "${WORKSPACE}/spring-petclinic-admin-server"
docker build --force-rm -t "${IMAGE_TAG_API_GATEWAY}" "${WORKSPACE}/spring-petclinic-api-gateway"
docker build --force-rm -t "${IMAGE_TAG_CONFIG_SERVER}" "${WORKSPACE}/spring-petclinic-config-server"
docker build --force-rm -t "${IMAGE_TAG_CUSTOMERS_SERVICE}" "${WORKSPACE}/spring-petclinic-customers-service"
docker build --force-rm -t "${IMAGE_TAG_DISCOVERY_SERVER}" "${WORKSPACE}/spring-petclinic-discovery-server"
docker build --force-rm -t "${IMAGE_TAG_HYSTRIX_DASHBOARD}" "${WORKSPACE}/spring-petclinic-hystrix-dashboard"
docker build --force-rm -t "${IMAGE_TAG_VETS_SERVICE}" "${WORKSPACE}/spring-petclinic-vets-service"
docker build --force-rm -t "${IMAGE_TAG_VISITS_SERVICE}" "${WORKSPACE}/spring-petclinic-visits-service"
docker build --force-rm -t "${IMAGE_TAG_GRAFANA_SERVICE}" "${WORKSPACE}/docker/grafana"
docker build --force-rm -t "${IMAGE_TAG_PROMETHEUS_SERVICE}" "${WORKSPACE}/docker/prometheus"
```

    • Imagelarimi push etmek icin script yazdim. jenkins>push-qa-docker-images-to-ecr.sh
```bash
aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
docker push "${IMAGE_TAG_ADMIN_SERVER}"
docker push "${IMAGE_TAG_API_GATEWAY}"
docker push "${IMAGE_TAG_CONFIG_SERVER}"
docker push "${IMAGE_TAG_CUSTOMERS_SERVICE}"
docker push "${IMAGE_TAG_DISCOVERY_SERVER}"
docker push "${IMAGE_TAG_HYSTRIX_DASHBOARD}"
docker push "${IMAGE_TAG_VETS_SERVICE}"
docker push "${IMAGE_TAG_VISITS_SERVICE}"
docker push "${IMAGE_TAG_GRAFANA_SERVICE}"
docker push "${IMAGE_TAG_PROMETHEUS_SERVICE}"
```
    • docker-compose-swarm-qa.yml adinda docker-compose file yazdim.
```yaml
version: '3.8'

services:
  config-server:
    image: "${IMAGE_TAG_CONFIG_SERVER}"
    deploy:
      resources:
        limits:
          memory: 512M #nightlu builden farkli olarak memory limit tedbir amacli eklenmis. 
    networks:
      - clarusnet
    ports:
     - 8888:8888

  discovery-server:
    image: "${IMAGE_TAG_DISCOVERY_SERVER}"
    deploy:
      resources:
        limits:
          memory: 512M
    depends_on:
      - config-server
    entrypoint: ["./dockerize","-wait=tcp://config-server:8888","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
     - 8761:8761

  customers-service:
    image: "${IMAGE_TAG_CUSTOMERS_SERVICE}"
    deploy:
      resources:
        limits:
          memory: 512M
      replicas: 3
      update_config:
          parallelism: 2
          delay: 5s
          order: start-first
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
    - 8081:8081

  visits-service:
    image: "${IMAGE_TAG_VISITS_SERVICE}"
    deploy:
      resources:
        limits:
          memory: 512M
      replicas: 3
      update_config:
          parallelism: 2
          delay: 5s
          order: start-first
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
     - 8082:8082

  vets-service:
    image: "${IMAGE_TAG_VETS_SERVICE}"
    deploy:
      resources:
        limits:
          memory: 512M
      replicas: 3
      update_config:
          parallelism: 2
          delay: 5s
          order: start-first
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
     - 8083:8083

  api-gateway:
    image: "${IMAGE_TAG_API_GATEWAY}"
    deploy:
      resources:
        limits:
          memory: 512M
      replicas: 5 #replica sayisini artirdim cunku bu surekli ayakta kalacak haftalik manuel test ortami onun icin her zaman ayakta olmasini istedigim icin replicayi yukselttim
      update_config:
          parallelism: 2
          delay: 5s
          order: start-first
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
     - 8080:8080

  tracing-server:
    image: openzipkin/zipkin
    deploy:
      resources:
        limits:
          memory: 512M
    environment:
    - JAVA_OPTS=-XX:+UnlockExperimentalVMOptions -Djava.security.egd=file:/dev/./urandom
    networks:
      - clarusnet
    ports:
     - 9411:9411

  admin-server:
    image: "${IMAGE_TAG_ADMIN_SERVER}"
    deploy:
      resources:
        limits:
          memory: 512M
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
     - 9090:9090

  hystrix-dashboard:
    image: "${IMAGE_TAG_HYSTRIX_DASHBOARD}"
    deploy:
      resources:
        limits:
          memory: 512M
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./dockerize","-wait=tcp://discovery-server:8761","-timeout=60s","--","java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    networks:
      - clarusnet
    ports:
     - 7979:7979

  ## Grafana / Prometheus

  grafana-server:
    image: "${IMAGE_TAG_GRAFANA_SERVICE}"
    deploy:
      resources:
        limits:
          memory: 256M
    networks:
      - clarusnet
    ports:
    - 3000:3000

  prometheus-server:
    image: "${IMAGE_TAG_PROMETHEUS_SERVICE}"
    deploy:
      resources:
        limits:
          memory: 256M
    networks:
      - clarusnet
    ports:
    - 9091:9090

networks:
  clarusnet:
    driver: overlay

    • Yukaridaki yaml file daki degiskenler ec2 larda gorunmuyor bu nedenle bu degiskenleri ec2 lara almak icin bir playbook yazdim. ansible>playbooks>pb_deploy_app_on_qa_environment.yaml olarak kaydettim.

---
- hosts: role_grand_master
  tasks:
  - name: Copy docker compose file to grand master
    copy:
      src: "{{ workspace }}/docker-compose-swarm-qa-tagged.yml" #docker-compose dosyasindaki degiskenleri bu playbookla birlikte yeni degiskenler olarak atayacagim ve bu dosyayi grand-master icinde calistirip bu dosya uzerinden docker-swarmi build edecem ve deploy edicem.
      dest: /home/ec2-user/docker-compose-swarm-qa-tagged.yml

  - name: get login credentials for ecr
    shell: "export PATH=$PATH:/usr/local/bin/ && aws ecr get-login-password --region {{ aws_region }} | docker login --username AWS --password-stdin {{ ecr_registry }}"
    register: output

  - name: deploy the app stack on swarm
    shell: "docker stack deploy --with-registry-auth -c /home/ec2-user/docker-compose-swarm-qa-tagged.yml {{ app_name }}" #uygulama bu dosya ile deploy olucak
    register: output

  - debug: msg="{{ output.stdout }}"
```
    • ansible>scripts>deploy_app_on_qa_environment.sh scriptini yaziyorum.
```bash
PATH="$PATH:/usr/local/bin"
APP_NAME="petclinic"
sed -i "s/APP_STACK_NAME/${APP_STACK_NAME}/" ./ansible/inventory/qa_stack_dynamic_inventory_aws_ec2.yaml
envsubst < docker-compose-swarm-qa.yml > docker-compose-swarm-qa-tagged.yml
#swarm-qa.yaml daki degiskenlerin degerlerini replace edip yeni bir dosya olusturuyor. Ansible playbook bu dosyayi alacak grand-masterin icine atacak ve build stack komutunu calistiracak 
ansible-playbook -i ./ansible/inventory/qa_stack_dynamic_inventory_aws_ec2.yaml -b --extra-vars "workspace=${WORKSPACE} app_name=${APP_NAME} aws_region=${AWS_REGION} ecr_registry=${ECR_REGISTRY}" ./ansible/playbooks/pb_deploy_app_on_qa_environment.yaml
```
    • Commit ve push

git add .
git commit -m 'added build scripts for QA Environment'
git push --set-upstream origin feature/msp-19
git checkout dev
git merge feature/msp-19
git push origin dev

# 20. MSP 20 - Build and Deploy App on QA Environment Manually
      
QA Setup for Release
Build and Deploy App on QA Environment Manually
MSP-20
Build and Deploy App for QA Environment Manually using Jenkins Jobs.
feature/msp-20


    • Bu asamada uygulamamizi deploy edecek ansible playbooku olusturdum.

    • jenkins>build-and-deploy-petclinic-on-qa-env-manually.sh scriptini olusturdum. 
```bash
PATH="$PATH:/usr/local/bin"
APP_NAME="petclinic"
APP_REPO_NAME="sezgin-repo/petclinic-app-qa"
APP_STACK_NAME="sezgin-petclinic-App-QA-1"
CFN_KEYPAIR="sezgin-petclinic-qa.key"
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
AWS_REGION="us-east-1"
ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
export ANSIBLE_PRIVATE_KEY_FILE="${JENKINS_HOME}/.ssh/${CFN_KEYPAIR}" //ssh klasorunu artik jenkins homedan aliyorum
export ANSIBLE_HOST_KEY_CHECKING="False"
echo 'Packaging the App into Jars with Maven'
. ./jenkins/package-with-maven-container.sh
echo 'Preparing QA Tags for Docker Images'
. ./jenkins/prepare-tags-ecr-for-qa-docker-images.sh
echo 'Building App QA Images'
. ./jenkins/build-qa-docker-images-for-ecr.sh
echo "Pushing App QA Images to ECR Repo"
. ./jenkins/push-qa-docker-images-to-ecr.sh
echo 'Deploying App on Swarm'
. ./ansible/scripts/deploy_app_on_qa_environment.sh
echo 'Deleting all local images'
docker image prune -af //imagelar jenkins servisde kalmasin bosuna yer kaplamasin jenkins serverdaki imagelar silinsin diye bu komutu koydum
```
    • Commit ve push.

git add .
git commit -m 'added script for jenkins job to build and deploy app on QA environment'
git push --set-upstream origin feature/msp-20
git checkout dev
git merge feature/msp-20
git push origin dev

    • releasei dev ile merge ediyorum cunku jenkins job release branchda calisacak.
      
git checkout release
git merge dev
git push origin release

    • Yukaridaki scriptimin dogru oldugunu kontrol ettim onun icin bir jenkins job yazdim. Jenkins>dashboard>newItem>freeStyleProject>build-and-deploy-petclinic-on-qa-env-manually.sh OK. Project URL yazdim. Release branch yazdim.
```bash
PATH="$PATH:/usr/local/bin"
APP_NAME="petclinic"
APP_REPO_NAME="sezgni-repo/petclinic-app-qa"
APP_STACK_NAME="sezgin-petclinic-App-QA-1"
CFN_KEYPAIR="sezgin-petclinic-qa.key"
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
AWS_REGION="us-east-1"
ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
export ANSIBLE_PRIVATE_KEY_FILE="${JENKINS_HOME}/.ssh/${CFN_KEYPAIR}" //ssh klasorunu artik jenkins homedan aliyorum
export ANSIBLE_HOST_KEY_CHECKING="False"
echo 'Packaging the App into Jars with Maven'
. ./jenkins/package-with-maven-container.sh
echo 'Preparing QA Tags for Docker Images'
. ./jenkins/prepare-tags-ecr-for-qa-docker-images.sh
echo 'Building App QA Images'
. ./jenkins/build-qa-docker-images-for-ecr.sh
echo "Pushing App QA Images to ECR Repo"
. ./jenkins/push-qa-docker-images-to-ecr.sh
echo 'Deploying App on Swarm'
. ./ansible/scripts/deploy_app_on_qa_environment.sh
echo 'Deleting all local images'
docker image prune -af //imagelar jenkins servisde kalmasin bosuna yer kaplamasin jenkins serverdaki imagelar silinsin diye bu komutu koydum
```
    • BUILD. Uygulama basari ile deploy edildi. 
      
# 21. MSP 21 - Prepare a QA Pipeline

QA Setup for Release
Prepare a QA Pipeline
MSP-21
Prepare a QA Pipeline using Jenkins on release branch for Weekly Builds.
feature/msp-21

    • Weekly pipeline ile environment a haftalik olarak uygulamam release branchindan alinacak ve haftalik olarak deploy edilcek boylece manuel testerlar bir hafta sure ile bu ortamda testlerini yapacaklar. Nightlyden farkli olarak imagelarin hepsi versiyonlanarak saklaniyor. Nightly pipelinedekiler ise siliniyor. Bu asamaya kadar manuel yaptik simdi bu asamaya kadar olan komutlari pipeline ile birlikte automatize hale getirdim.
      
    • feature ekledim.

git checkout dev
git branch feature/msp-21
git checkout feature/msp-21

    • jenkins>jenkinsfile-petclinic-weekly-qa scriptini yazdim.
```groovy
pipeline {
    agent { label "master" }
    environment {
        PATH=sh(script:"echo $PATH:/usr/local/bin", returnStdout:true).trim()
        APP_NAME="petclinic"
        APP_REPO_NAME="sezgin-repo/petclinic-app-qa"
        APP_STACK_NAME="sezgin-petclinic-App-QA-1"
        CFN_KEYPAIR="sezgin-petclinic-qa.key"
        AWS_ACCOUNT_ID=sh(script:'export PATH="$PATH:/usr/local/bin" && aws sts get-caller-identity --query Account --output text', returnStdout:true).trim()
        AWS_REGION="us-east-1"
        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ANSIBLE_PRIVATE_KEY_FILE="${JENKINS_HOME}/.ssh/${CFN_KEYPAIR}"
        ANSIBLE_HOST_KEY_CHECKING="False"
    }
//application packaging
    stages {
        stage('Package Application') {
            steps {
                echo 'Packaging the app into jars with maven'
                sh ". ./jenkins/package-with-maven-container.sh"
            }
        }
//olusturulan jar dosyalarini imagea koymak icin image scriptlerini yazdim
        stage('Prepare Tags for Docker Images') {
            steps {
                echo 'Preparing Tags for Docker Images'
                script {
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    env.IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
                    env.IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
                }
            }
        }
//tagleri olusturduktan sonra tagleri build ediyorum
        stage('Build App Docker Images') {
            steps {
                echo 'Building App Dev Images'
                sh ". ./jenkins/build-qa-docker-images-for-ecr.sh"
                sh 'docker image ls'
            }
        }
//ecr reposuna push ettim
        stage('Push Images to ECR Repo') {
            steps {
                echo "Pushing ${APP_NAME} App Images to ECR Repo"
                sh ". ./jenkins/push-qa-docker-images-to-ecr.sh"
            }
        }
//zaten arkada hazir olan docker swarm environmentina uygulamami deploy ettim
        stage('Deploy App on Docker Swarm'){
            steps {
                echo 'Deploying App on Swarm'
                sh '. ./ansible/scripts/deploy_app_on_qa_environment.sh'
            }
        }
    }
//localimde yer kaplamasin diye localimdeki imagelari sildim
    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
        }
    }
}
```

    • Commit push.

git add .
git commit -m 'added jenkinsfile petclinic-weekly-qa for release branch'
git push --set-upstream origin feature/msp-21
git checkout dev
git merge feature/msp-21
git push origin dev

    • dev ile merge ediyorum

git checkout release
git merge dev
git push origin release

    • jenkins>dashboard>pipeline>petclinic-weekly-qa OK. Build-periodically>59 23 * * 0. Pipeline>Project URL yapistir. Brances>*/release. Script Path>jenkins/jenkinsfile-oetclinic-weekly-qa yazdim.
    • BUILD. Docker swarm a basarili bir sekilde uygulama deploy edildi. Manuel testerlar icin environment hazir bir haftalik test yapmak icin hazir.

# 22. MSP 22 - Prepare Petlinic Kubernetes YAML Files

Staging and Production Setup
Prepare Petlinic Kubernetes YAML Files
MSP-22
Prepare Petlinic Kubernetes YAML Files for Staging and Production Pipelines.
feature/msp-22

    • feature ekledim.
      
git checkout release
git branch feature/msp-22
git checkout feature/msp-22

    • k8s folderi olusturdum.
    • K8s nin altinda docker-compose.yml fileini olusturdum. Docker-Compose.yaml dan k8s olusturdum. Deployment ve ingresse ihtiyacim var.

```yaml
version: '3'
services:
  config-server:
    image: IMAGE_TAG_CONFIG_SERVER
    ports:
     - 8888:8888
    labels:
      kompose.image-pull-secret: "regcred " #secret olusturuyor. Imagelarimiz ecr repodan aldigimiz icin k8s e bilgi vermemiz gerekiyor ve secret olusturmamiz gerekiyor. Burada diyoruz ki bunu oraya koy ben de secret olusturdugumda otomatik olarak onu goreyim. Regcred isminin bir anlami yok. Kendim yazdim farkli olabilirdi,
  discovery-server:
    image: IMAGE_TAG_DISCOVERY_SERVER
    ports:
     - 8761:8761
    labels:
      kompose.image-pull-secret: "regcred"
  customers-service:
    image: IMAGE_TAG_CUSTOMERS_SERVICE
    deploy:
      replicas: 2
    ports:
    - 8081:8081
    labels:
      kompose.image-pull-secret: "regcred"
      kompose.service.expose: "drsezginerdem.com" #k8s icin ingress olusturuyor
  visits-service:
    image: IMAGE_TAG_VISITS_SERVICE
    deploy:
      replicas: 2
    ports:
     - 8082:8082
    labels:
      kompose.image-pull-secret: "regcred"
      kompose.service.expose: "drsezginerdem.com"
  vets-service:
    image: IMAGE_TAG_VETS_SERVICE
    deploy:
      replicas: 2
    ports:
     - 8083:8083
    labels:
      kompose.image-pull-secret: "regcred"
      kompose.service.expose: "drsezginerdem.com"
  api-gateway:
    image: IMAGE_TAG_API_GATEWAY
    deploy:
      replicas: 2
    ports:
     - 8080:8080
    labels:
      kompose.image-pull-secret: "regcred"
      kompose.service.expose: "drsezginerdem.com"
  tracing-server:
    image: openzipkin/zipkin
    environment:
    - JAVA_OPTS=-XX:+UnlockExperimentalVMOptions -Djava.security.egd=file:/dev/./urandom
    ports:
     - 9411:9411
  admin-server:
    image: IMAGE_TAG_ADMIN_SERVER
    ports:
     - 9090:9090
    labels:
      kompose.image-pull-secret: "regcred"
  hystrix-dashboard:
    image: IMAGE_TAG_HYSTRIX_DASHBOARD
    ports:
     - 7979:7979
    labels:
      kompose.image-pull-secret: "regcred"
  grafana-server:
    image: IMAGE_TAG_GRAFANA_SERVICE
    ports:
    - 3000:3000
    labels:
      kompose.image-pull-secret: "regcred"
  prometheus-server:
    image: IMAGE_TAG_PROMETHEUS_SERVICE
    ports:
    - 9091:9090
    labels:
      kompose.image-pull-secret: "regcred"
```

    • conversion tool olarak Kompose yukluyorum. Bu tool sayesinde docker-compose.yml filei k8s ihtiyacimiz olan objelerin yml filelerini veriyor.

curl -L https://github.com/kubernetes/kompose/releases/download/v1.22.0/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose
kompose version


    • k8s altinda base, staging, prod klasorleri olusturdum.
    • k8s/base inin altina geldim.
```bash
kompose convert -f k8s/docker-compose.yml -o k8s/base #docker-compose.yml filenin donustur ve k8s klasorunun icine at.
```
    • Mikroservice leri sirasiyla baslatmak icin init-containers ile deployment files lari guncelledim. Docker-swarmda depends on vardi veya dockerize ile calisip calismadigini bekliyorduk. Bunu da k8s de init container ile yapiyoruz. Butun k8s/base klasorunun altindaki deployment yaml lari guncelledim. Sadece config serverda birsey olmayacak cunku o ilk olusuyor. Kimseyi beklemiyor.
```bash
# for discovery server #sadece discovery icin bunu
      initContainers:
      - name: init-config-server
        image: busybox
        command: ['sh', '-c', 'until nc -z config-server:8888; do echo waiting for config-server; sleep 2; done;'] #nc cat komutu demek, config serverden 8888 I dinle until e kadar sonra olursa done ciktisini yazdir.
# for all other microservices except config-server and discovery-server #digerlerinin hepsi icin bunu
      initContainers:
      - name: init-discovery-server
        image: busybox
        command: ['sh', '-c', 'until nc -z discovery-server:8761; do echo waiting for discovery-server; sleep 2; done;']
```

    • customers-service-ingress.yaml fileinin guncelledim. Ilgili kisimlari guncelliyorum. Tum dosyasinin icerigi bu sekilde olmayacak.

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: petclinic.clarusway.us
    http:
      paths:
      - backend:
          serviceName: customers-service
          servicePort: 8081
        path: /api/gateway(/|$)(.*)
      - backend:
          serviceName: customers-service
          servicePort: 8081
        path: /api/customer(/|$)(.*)
```

    • visits-service-ingress.yaml fileini guncelledim. Ilgili kisimlari guncelliyorum. Tum dosyanin icerigi bu sekilde olmayacak.

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: petclinic.clarusway.us
    http:
      paths:
      - backend:
          serviceName: visits-service
          servicePort: 8082
        path: /api/visit(/|$)(.*)
```

    • vets-service-ingress.yaml fileini guncelledim. Ilgili kisimlari guncelliyorum. Tum dosyasinin icerigi bu sekilde olmayacak.

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: petclinic.clarusway.us
    http:
      paths:
      - backend:
          serviceName: vets-service
          servicePort: 8083
        path: /api/vet(/|$)(.*)
```
    • docker-compose file imiz daha onceden hazirdi ben bunu k8s e cevirmek istiyorum. Bunu zaten kompose convert komutu ile onceden yapmistim bunda bir sorun yok. Kustomazation tool u da ayri bir tool. kustomization.yml dosyasi icerisine koydugum resources dosyasinda kustomization.yml dosyasi icinde belirttigim degisiklikleri yapiyor bu degisiklikleri kubectl kustomize ./ komutu ile gorebilirim ama degisikligi uygulamam icin kubectl apply -k ./ komutu ile yapiyorum. Bir diger yontem de compozing. kustomization.yml icerisine resources olarak iki tane dosya yazdigimda ise kubectl apply -k dedigimde bu iki dosyayi ayni anda calistirmis oluyorum. Tek bir komutla birden fazla kaynagi yonetmeme imkan veriyor. Dosyanin icerisindeki alanlari degistiriyor. Dosya orjinal olarak kaliyor sadece degistirip yaratiyor dosya orjinal kaliyor. kustomization.yml da patches ise degisikligi nasil yapacagimi baska bir yaml dosyasi uzerinden gosteriyor.
    • Envsubst komutu: envsubst < ./envsubst-test > ./new-envsubst-test  bu komutla /envsubst-test dosyasinin icindeki keylerin value larini ./new-envsubst-test dosyasina yazdiriyor

    • Base>kustomization-template.yml isimli filei koy. Asagidaki imagelari servicelerin deployment dosyalarindan aliyor. Imagelardaki name imagein isminin eski hali ben bu ismi her build ettikten sonra degistirmek istiyorum. Yeni imagain ismini bu tool sayesinde deployment dosyalarina koymak istiyorum. 

```yaml
resources:
- admin-server-deployment.yaml
- api-gateway-deployment.yaml
- config-server-deployment.yaml
- customers-service-deployment.yaml
- discovery-server-deployment.yaml
- grafana-server-deployment.yaml
- hystrix-dashboard-deployment.yaml
- prometheus-server-deployment.yaml
- tracing-server-deployment.yaml
- vets-service-deployment.yaml
- visits-service-deployment.yaml
- admin-server-service.yaml
- api-gateway-service.yaml
- config-server-service.yaml
- customers-service-service.yaml
- discovery-server-service.yaml
- grafana-server-service.yaml
- hystrix-dashboard-service.yaml
- prometheus-server-service.yaml
- tracing-server-service.yaml
- vets-service-service.yaml
- visits-service-service.yaml
- api-gateway-ingress.yaml
- customers-service-ingress.yaml
- vets-service-ingress.yaml
- visits-service-ingress.yaml

images:
- name: IMAGE_TAG_CONFIG_SERVER
  newName: "${IMAGE_TAG_CONFIG_SERVER}"
- name: IMAGE_TAG_DISCOVERY_SERVER
  newName: "${IMAGE_TAG_DISCOVERY_SERVER}"
- name: IMAGE_TAG_CUSTOMERS_SERVICE
  newName: "${IMAGE_TAG_CUSTOMERS_SERVICE}"
- name: IMAGE_TAG_VISITS_SERVICE
  newName: "${IMAGE_TAG_VISITS_SERVICE}"
- name: IMAGE_TAG_VETS_SERVICE
  newName: "${IMAGE_TAG_VETS_SERVICE}"
- name: IMAGE_TAG_API_GATEWAY
  newName: "${IMAGE_TAG_API_GATEWAY}"
- name: IMAGE_TAG_ADMIN_SERVER
  newName: "${IMAGE_TAG_ADMIN_SERVER}"
- name: IMAGE_TAG_HYSTRIX_DASHBOARD
  newName: "${IMAGE_TAG_HYSTRIX_DASHBOARD}"
- name: IMAGE_TAG_GRAFANA_SERVICE
  newName: "${IMAGE_TAG_GRAFANA_SERVICE}"
- name: IMAGE_TAG_PROMETHEUS_SERVICE
  newName: "${IMAGE_TAG_PROMETHEUS_SERVICE}"
```
    • k8s/staging/kustomization.yml ekledim. Namespace ekle. Sonra base klasoru icine replica-count yamli bul dedim. Bundakilere bakarak kustomize yaptim.

```yaml
# kustomization.yml
namespace: "petclinic-staging-ns"
bases:
- ../base
patches:
- replica-count.yml
```
    • k8s/staging/replica-count.yml filelarini ekledim. Bastaki bilgiler apiversion, kind, metadata sonra degistirilecek kismi giriyorum. Bu bilgileri ilgili yerde kustomize ediyorum

```yaml
# replica-count.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
spec:
  replicas: 3

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: api-gateway
spec:
  rules:
    - host: drsezginerdem.com
      http:
        paths:
          - backend:
              serviceName: api-gateway
              servicePort: 8080

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: customers-service
spec:
  rules:
  - host: petclinic.clarusway.us
    http:
      paths:
      - backend:
          serviceName: customers-service
          servicePort: 8081
        path: /api/gateway(/|$)(.*)
      - backend:
          serviceName: customers-service
          servicePort: 8081
        path: /api/customer(/|$)(.*)

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: vets-service
spec:
  rules:
  - host: drsezginerdem.com
    http:
      paths:
      - backend:
          serviceName: vets-service
          servicePort: 8083
        path: /api/vet(/|$)(.*)

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: visits-service
spec:
  rules:
  - host: drsezginerdem@gmail.com
    http:
      paths:
      - backend:
          serviceName: visits-service
          servicePort: 8082
        path: /api/visit(/|$)(.*)
```
    • k8s/prod/kustomization.yml dosyasini koydum.#pod icindeki kustomization dosyami kontrol ettim. Kustomization komutu ne yapiyor k8s/prod daki kustomization.yml dosyasina bakiyor. Bu dosyanin icinde asagidaki yaml var. . ../base e bak onun icindeki kustomization.yml I bul.  Oradaki resources lara bak onlarin her birisine namespace ekle. Bunu yaparak bunun ciktisini da ekrana cikariyor.

```yaml
# kustomization.yml
namespace: "petclinic-prod-ns"
bases:
- ../base
patches:
- replica-count.yml
```
    • k8s/prod/replica-count.yml dosyasini koydum. Api-gateway in replicasini 5 yap. 

```yaml
# replica-count.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
spec:
  replicas: 5
```

    • kubectl I yukledim ve setup ini yaptim.

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.18.9/2020-11-02/bin/linux/amd64/kubectl
sudo mv kubectl /usr/local/bin/kubectl
chmod +x /usr/local/bin/kubectl
kubectl version --short --client
```
    • kustomization. Tool unun calistigini kontrol ediyorum. Export ederek env variable giriyorum. 

```bash
#export ile env variable giriyorum.

export IMAGE_TAG_CONFIG_SERVER="testing-image-1"    
export IMAGE_TAG_DISCOVERY_SERVER="testing-image-2" 
export IMAGE_TAG_CUSTOMERS_SERVICE="testing-image-3"
export IMAGE_TAG_VISITS_SERVICE="testing-image-4"   
export IMAGE_TAG_VETS_SERVICE="testing-image-5"     
export IMAGE_TAG_API_GATEWAY="testing-image-6"      
export IMAGE_TAG_ADMIN_SERVER="testing-image-7"     
export IMAGE_TAG_HYSTRIX_DASHBOARD="testing-image-8"
export IMAGE_TAG_GRAFANA_SERVICE="testing-image-9"
export IMAGE_TAG_PROMETHEUS_SERVICE="testing-image-10"
# create base kustomization file from template by updating with environments variables #bu komutla kustomization-template.yml icindeki envsubst lari kustomization.yml icine at dedim
envsubst < k8s/base/kustomization-template.yml > k8s/base/kustomization.yml
# test customization for production 
kubectl kustomize k8s/prod/
# test customization for staging #ayni islemi staging icin de yaptim
kubectl kustomize k8s/staging/
```
    • Commit ve Push

git add .
git commit -m 'added Configuration YAML Files for Kubernetes Deployment'
git push --set-upstream origin feature/msp-22
git checkout release
git merge feature/msp-22
git push origin release

# 23. MSP 23 - Prepare High-availability RKE Kubernetes Cluster on AWS EC2

Staging and Production Setup
Prepare HA RKE Kubernetes Cluster
MSP-23
Prepare High-availability RKE Kubernetes Cluster on AWS EC2
feature/msp-23


    • Rancher: Bu bir server. Nasil Jenkins server vasitasiyla AWS deki infrastructure imizi yonetiyoruz bunun sayesinde de kubernetes clusterimizi yonetiyoruz. Hem rancher vasitasiyla kuruyorum hem de yonetiyorum. Istersem birden fazla yerde k8s imizi rancher vasitasiyla yonetebilirim. Ben su an bir server icine rancher kurdum maliyet olmasin ve egitim olsun diye ancak gercek ortamda 3 adet ec2 instance uzerine 1 rancher kurarim yani biri cokerse digerleri ile bunu yedeklerim. Rancher benim managerim olacak islemlerimi onun uzerinden yapacagim.
      
    • feature ekledim.

git checkout release
git branch feature/msp-23
git checkout feature/msp-23

    • infrastructure/sezgin-rke-controlplane-policy.json policesini ekledim. Bunlari amazonda olusturacagim ama kaybolmasin diye buraya koydum. Rancher in amazondaki ihtiyac duydugu yetkileri verdim. Tek bir rancher serverimiz oldugu icin manager yetkilerini de buna verecegim worker yetkilerini de buna verecegim. Eger 3 instance olsa idi birine manager digerlerine worker yetkileri verirdim.
```js  
{
"Version": "2012-10-17",
"Statement": [
  {
    "Effect": "Allow",
    "Action": [
      "autoscaling:DescribeAutoScalingGroups",
      "autoscaling:DescribeLaunchConfigurations",
      "autoscaling:DescribeTags",
      "ec2:DescribeInstances",
      "ec2:DescribeRegions",
      "ec2:DescribeRouteTables",
      "ec2:DescribeSecurityGroups",
      "ec2:DescribeSubnets",
      "ec2:DescribeVolumes",
      "ec2:CreateSecurityGroup",
      "ec2:CreateTags",
      "ec2:CreateVolume",
      "ec2:ModifyInstanceAttribute",
      "ec2:ModifyVolume",
      "ec2:AttachVolume",
      "ec2:AuthorizeSecurityGroupIngress",
      "ec2:CreateRoute",
      "ec2:DeleteRoute",
      "ec2:DeleteSecurityGroup",
      "ec2:DeleteVolume",
      "ec2:DetachVolume",
      "ec2:RevokeSecurityGroupIngress",
      "ec2:DescribeVpcs",
      "elasticloadbalancing:AddTags",
      "elasticloadbalancing:AttachLoadBalancerToSubnets",
      "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer",
      "elasticloadbalancing:CreateLoadBalancer",
      "elasticloadbalancing:CreateLoadBalancerPolicy",
      "elasticloadbalancing:CreateLoadBalancerListeners",
      "elasticloadbalancing:ConfigureHealthCheck",
      "elasticloadbalancing:DeleteLoadBalancer",
      "elasticloadbalancing:DeleteLoadBalancerListeners",
      "elasticloadbalancing:DescribeLoadBalancers",
      "elasticloadbalancing:DescribeLoadBalancerAttributes",
      "elasticloadbalancing:DetachLoadBalancerFromSubnets",
      "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
      "elasticloadbalancing:ModifyLoadBalancerAttributes",
      "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
      "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer",
      "elasticloadbalancing:AddTags",
      "elasticloadbalancing:CreateListener",
      "elasticloadbalancing:CreateTargetGroup",
      "elasticloadbalancing:DeleteListener",
      "elasticloadbalancing:DeleteTargetGroup",
      "elasticloadbalancing:DescribeListeners",
      "elasticloadbalancing:DescribeLoadBalancerPolicies",
      "elasticloadbalancing:DescribeTargetGroups",
      "elasticloadbalancing:DescribeTargetHealth",
      "elasticloadbalancing:ModifyListener",
      "elasticloadbalancing:ModifyTargetGroup",
      "elasticloadbalancing:RegisterTargets",
      "elasticloadbalancing:SetLoadBalancerPoliciesOfListener",
      "iam:CreateServiceLinkedRole",
      "kms:DescribeKey"
    ],
    "Resource": [
      "*"
    ]
  }
]
}
```
    • infrastructure/sezgin-rke-etcd-worker-policy.json policesini ekledim.
```js
{
"Version": "2012-10-17",
"Statement": [
    {
        "Effect": "Allow",
        "Action": [
            "ec2:DescribeInstances",
            "ec2:DescribeRegions",
            "ecr:GetAuthorizationToken",
            "ecr:BatchCheckLayerAvailability",
            "ecr:GetDownloadUrlForLayer",
            "ecr:GetRepositoryPolicy",
            "ecr:DescribeRepositories",
            "ecr:ListImages",
            "ecr:BatchGetImage"
        ],
        "Resource": "*"
    }
]
}
```
    • AWS console dan IAM servisine girdim. Create policy>JSON>sezgin-rke-controlplane-policy  >review policy>name>sezgin-rke-controlplane-policy>create policy yaparak policy olusturdum.
    • AWS console dan IAM servisine girdim. Create policy>JSON>sezgin-rke-controlplane-policy  >review policy>name>sezgin-rke-etcd-worker-policy>create policy yaparak policy olusturdum.
    • AWS console dan roles>create role>EC2>next>ezgin-rke-controlplane-policy, sezgin-rke-controlplane-policy>next>next>rolename>sezgin-rke-role>next role olusturuldu.
    • Load balancer icin bir security group yaratacagim. Console>ec2>create security group>name>sezgin-rke-alb-sg>defauly vpc>inbound rules> hhtp, https-anywhere>create security group.
    • Rancher icin bir security group yaratacagim. Console>ec2>create security group>name>sezgin-rke-cluster-sg>defauly vpc>inbound rules> hhtp anywhere, https anywhere, ssh da customin yaninda alb nin sg ekledim, custom tcp 6443 custom in yaninda alb nin sg ekledim>create security group.
    • Bir rancher icin bir server da olabilir 3 tane rancher da olabilir. Aslinda su an bir egitim oldugu icin rancher clusteri olmadigi icin tek bir rancher server oldugu icin buna gerek yok ama gormek acisindan bunu da ekledim. Tekrar sezgin-rke-cluster-sg giriyorum edit inbound>all traffic>custom in yanina sezgin-rke-cluster-sg>save rules.
    • Outbound ruleslar da asagida.
    • Yapilanlarin aciklamasi asagida

Create a security group for RKE Kubernetes Cluster with name of call-rke-cluster-sg and define following inbound and outbound rules.

Inbound rules;

Allow HTTP protocol (TCP on port 80) from Application Load Balancer.

Allow HTTPS protocol (TCP on port 443) from any source that needs to use Rancher UI or API.

Allow TCP on port 6443 from any source that needs to use Kubernetes API server(ex. Jenkins Server).

Allow SSH on port 22 to any node IP that installs Docker (ex. Jenkins Server).

Outbound rules;

Allow SSH on port 22 to any node IP from a node created using Node Driver.

Allow HTTPS protocol (TCP on port 443) to 35.160.43.145/32, 35.167.242.46/32, 52.33.59.17/32 for catalogs of git.rancher.io.

Allow TCP on port 2376 to any node IP from a node created using Node Driver for Docker machine TLS port.

    • Rancher a jenkins serverdan baglanabilmek icin bir keypair yarattim.
```bash
aws ec2 create-key-pair --region us-east-1 --key-name sezgin-rancher.key --query KeyMaterial --output text > ~/.ssh/sezgin-rancher.key
chmod 400 ~/.ssh/sezgin-rancher.key
```
    • Rancher icin Ubuntu Server 20.04 LTS (HVM) ami-0885b1f6bd170450c (64-bit x86) with t2.medium type, 16 GB root volume, call-rke-cluster-sg security group, sezgin-rke-role IAM Role, Name:Sezgin-Rancher-Cluster-Instance tag, Key = kubernetes.io/cluster/sezgin-Rancher and Value = owned olan baska bir key daha ekledim and sezgin-rancher.key key-pair. 
    •  Rancher dokumantasyonundan bakildiginda Amazona rancher kurulacaksa bu taglari atamamiz gerektigi zaten yaziyor. Bu tagler sayesinde rancher nodelari takip ediyor. EC2 nun Subnet ve security-group icin tag atiyorum Key = kubernetes.io/cluster/sezgin-Rancher and Value = owned. Security groups>sezgin-rke-cluster-sg>tags> manage tags>key= kubernetes.io/cluster/sezgin-Rancher, Value = owned>SAVE. subnets>tags>add new tag> Key = kubernetes.io/cluster/sezgin-Rancher ve Value = owned>SAVE.
    • Jenkis server a girip docker yukleyecegim. Neden docker yuklemem gerekir Rancher serverimiza rancher kubernetes engine kurmamiz icin dockera ihtiyamiz var.
    • Jenkins serveri bastion host olarak kullanip rancher a baglaniyorum.
      ssh -i “sezgin-rancher.key” ubuntu@<ip adresi>.compute.1.amazonaws.com
    • docker yukluyorum.

## Set hostname of instance
sudo hostnamectl set-hostname rancher-instance-1
## Update OS 
sudo apt-get update -y
sudo apt-get upgrade -y
## Install and start Docker on Ubuntu 20.04
sudo apt install docker.io -y  
sudo systemctl start docker
sudo systemctl enable docker
## Add ubuntu user to docker group
sudo usermod -aG docker ubuntu
newgrp docker
## Docker control
docker version

    • ALB kuracagim ama bunun icin bir target gruba ihtiyacim var. AWS Console>target groups>create target group>groupname>sezgin-rancher-http-80-tg>path>/healths>healthy treshold 3>unhealthy threshold 3>interal 10 NEXT sezgin-rancher-cluster-instance secilecek>include as pending below>create target group. Boylelikle target group olustu,

Target type         : instance
Protocol            : HTTP
Port                : 80

<!-- Health Checks Settings -->
Protocol            : HTTP
Path                : /healthz
Port                : traffic port
Healthy threshold   : 3
Unhealthy threshold : 3
Timeout             : 5 seconds
Interval            : 10 seoconds
Success             : 200


    • Load balancer olusturmak icin AWS Console>load balancer>create load balancer>application load balancer>name>sezgin-rancher-alb>add listener>https > rancher serverin bulundugu region ve bir tane daha region sec>next>certificate name> next>sezgin-rke-alb-sg>target group>existing target group>name>sezgin-rancher-http-80-tg>next>review>CREATE

Scheme              : internet-facing
IP address type     : ipv4

<!-- Listeners-->
Protocol            : HTTPS/HTTP
Port                : 443/80
Availability Zones  : Select AZs of RKE instances
Target group        : `sezgin-rancher-http-80-tg` target group 

      * Certificate yoksa certificate almak icin ACM (Amazon Certificate Manager)>domain name>*.drsezginerdem.com>next>DNS validation>next>key=name value=sezgin>CONFIRM>CONTINUE.
    • Load balancerimi configure ettim. AWS Console>Load Balancer>listeners>HTTP 80>edit>forward to delete> add action>redirect>https>443>update
    • Route 53>hosted zone>create record>simple record>define simple record>rancher>alian to aplication and classic load balancer>us-east-1>route type>A-Route traffic to an IP4 address and some AWS resources>define simple record>create records.
    • RKE kubernetes kurmak icin bir tool. Bunun vasitasiyla kubernetes kuracagim. Docker kurdum zaten. Rancher severa geldim. RKE toolunu kuruyorum.

curl -SsL "https://github.com/rancher/rke/releases/download/v1.1.12/rke_linux-amd64" -o "rke_linux-amd64"
sudo mv rke_linux-amd64 /usr/local/bin/rke
chmod +x /usr/local/bin/rke
rke --version

    • rancher servere kubernetes kurdum. infrastructure>rancher-cluster.yml ekledim.

```yaml
- address: 18.132.196.64 #rancher in public IPsi
  internal_address: 172.31.30.252 #rancher in private IPsi
  user: ubuntu 
  role: [controlplane, worker, etcd] #burada bir cluster kuruyoruz normalde ama biz tek bir instance a kurdugumuz icin hepsini tek bir role e atadik. Bunlarin hepsi ayri bir node olurdu gercek bir projede

services:
  etcd:
    snapshot: true #6 saatte bir snapshot al 24 saat sonra sil
    creation: 6h
    retention: 24h

ssh_key_path: ~/.ssh/sezgin-rancher.key

# Required for external TLS termination with
# ingress-nginx v0.22+
# load balancer kurdugumuz icin burasinin ayari bu sekilde olmali
ingress:
  provider: nginx
  options:
    use-forwarded-headers: "true"
```

    • Rancher instance>security groups>edit inbound>add rule>ssh ve custom tcp(6443) ikisine de custom in yanina da jenkins server imin public ip adresini giriyorum.
    • Kubernetes cluster kurmak icin Infrastructure klasorumun icinde asagidaki komutu girdim. RKE tool u jenkins severindan rancer serverina kubernetes clusterinin kurulmunu sagliyor.

```bash
rke up --config ./rancher-cluster.yml
```

    • Kubernetes clusterinin kurulumunun basarili olup olmadigini kontrol ediyorum.

```bash
mkdir -p ~/.kube
mv ./kube_config_rancher-cluster.yml $HOME/.kube/config #bu komutla kube_config-cluster.yml dosyasini ./kube/config in icine tasiyorum ki kubectl komutlarini girdigimde oradan okudugu icin anlayabilsin.
chmod 400 ~/.kube/config
kubectl get nodes
kubectl get pods --all-namespaces
```
    • Kubernetes kurulumu tamamlandi ama rancher kurulumu tamamlanmadi. Normalde bunlari githubda tutmayiz ancak bu egitim maksatli oldugu icin tutuyoruz. Git ignore ile bunlari gonderme diyebiliriz.
    • Commit and change

git add .
git commit -m 'added rancher setup files'
git push --set-upstream origin feature/msp-23
git checkout release
git merge feature/msp-23
git push origin release

# 24. MSP 24 - Install Rancher App on RKE Kubernetes Cluster

Staging and Production Setup
Install Rancher App on RKE K8s Cluster
MSP-24
Install Rancher App on RKE Kubernetes Cluster

    • Rancher kubernetes cluster ina rancher-app yukleyecegim.

    • Eger bu stageden once instancelari kapatti isem infrastructure/rancher-cluster.yaml dosyasina gelip oradan private ip leri guncellemem gerekiyior. Sonrasinda infrastructure klasorunun icinde

```bash
rke down --config ./rancher-cluster.yml
```

komutu ile clusteri sokuyorum ve daha sonra yeniden

```bash
rke up --config ./rancher-cluster.yml
```

komutu ile clusteri yeniden ayaga kaldiriyorum. Eger instancelari kapatip acmadi isem bu islemleri yapmama gerek yok. Asagidaki komutu yeniden girmem gerekiyor

```bash
mv ./kube_config_rancher-cluster.yml $HOME/.kube/config
```

    • Helm kurulumu icin asagidaki komutu giriyorum

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm version
```

    • Helm nedir? Yum gibi apt gibi package yonetim sistemidir. Chart bir nevi image, repository imagelarin bulundugu yer, release ise charti kurdugumuz calistirdigimiz sey bir nevi containerdir. Hemlerin bulundugu docker-hub gibi bir repository var.
    • Rancherin chartini indiriyorum ve helm repolari listeliyorum.

```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo list
```

    • Namespace kuruyorum.

```bash
kubectl create namespace cattle-system
```

    • Rancher yukluyorum.

```bash
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.onecard.click \
  --set tls=external \
  --set replicas=1
```

    • Rancher server basarili bir bicimde deploy edildi mi kontrol ediyorum.

```bash
kubectl -n cattle-system get deploy rancher
kubectl -n cattle-system get pods
```

    • rancher.drsezginerdem.com adresinden rancher servera ulastim.

# 25. MSP 25 - Create Staging and Production Environment with Rancher

Staging and Production Setup
Create Staging and Production Environment with Rancher
MSP-25
Create Staging and Production Environment with Rancher by creating new cluster for Petclinic

    • new Password>confirm Password>manage multiple clusters>allow collection>I agree>CONTINUE eger hata olursa helm uninstall ile tekrar kaldirip kurabilirim.
    • Rancherin AWS deki resourcelarimi yonetmesi icin Cloud credential tanimlayacagim. Sagda kosedeki isarette>cloud credential>add cloud credential>access key ve secret keyleri girdim>name>sezgin-aws-training-account.
    • Ayni yerden sagdaki ust koseden>Node templates>add template>amazon ec2>cloud credential>sezgin-aws-training-account>next>default vpc>select security group>standard>instance ozellikleri asagidaki gibi olacak>name>sezgin>key=os>value=rancheros OK diyerek tamamliyorum.
      
Region            : us-east-1
Security group    : create new sg (rancher-nodes)
Instance Type     : t2.medium
Root Disk Size    : 16 GB
AMI (RancherOS)   : ami-0e8a3347e4c5959bd #rancherOS AMI kullaniyoruz
SSH User          : rancher
Label             : os=rancheros

# 26. MSP 26 - Prepare a Staging Pipeline

Staging Deployment Setup
Prepare a Staging Pipeline
MSP-26
Prepare a Staging Pipeline on Jenkins Server
feature/msp-26

    • Yeni feature

git checkout release
git branch feature/msp-26
git checkout feature/msp-26

    • Rancher da RKE kullanarak petclinic-cluster-staging adinda bir kubernetes clusteri olusturuyorum.
    • rancher>clusters>add cluster>ec2>clustername>petclinic-cluster-staging>name prefix>petclinic-k8s-instance>count 3>CREATE

Cluster Type      : Amazon EC2
Name Prefix       : petclinic-k8s-instance
Count             : 3
etcd              : checked
Control Plane     : checked
Worker            : checked

    • Consola gidip 3 adet instance olusup olmadigini kontrol ediyorum. 
    • Petclinic-cluster-staging icinde namespace olusturdum. Petclinic-cluster-staging>Project/namespaces>Project Default>add namespace>petclinic-staging-ns>CREATE
    • ECR repository olusturdum. Jenkins Dashboard>new item>freestyle Project>reate-ecr-docker-registry-for-petclinic-staging>OK>execute shell>
```bash
PATH="$PATH:/usr/local/bin"
APP_REPO_NAME="sezgin-repo/petclinic-app-staging"
AWS_REGION="us-east-1"

aws ecr create-repository \
  --repository-name ${APP_REPO_NAME} \
  --image-scanning-configuration scanOnPush=false \
  --image-tag-mutability MUTABLE \
  --region ${AWS_REGION}
```
    • BUILD ve repository olustu.
    • Imagelarima tag atadim bunun icin bir scriptt yazdim. jenkins/prepare-tags-ecr-for-staging-docker-images.sh

```bash
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
export IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
export IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
```

    • Imagelarimi build etmek icin bir script yazdim. jenkins/build-staging-docker-images-for-ecr.sh
```bash
docker build --force-rm -t "${IMAGE_TAG_ADMIN_SERVER}" "${WORKSPACE}/spring-petclinic-admin-server"
docker build --force-rm -t "${IMAGE_TAG_API_GATEWAY}" "${WORKSPACE}/spring-petclinic-api-gateway"
docker build --force-rm -t "${IMAGE_TAG_CONFIG_SERVER}" "${WORKSPACE}/spring-petclinic-config-server"
docker build --force-rm -t "${IMAGE_TAG_CUSTOMERS_SERVICE}" "${WORKSPACE}/spring-petclinic-customers-service"
docker build --force-rm -t "${IMAGE_TAG_DISCOVERY_SERVER}" "${WORKSPACE}/spring-petclinic-discovery-server"
docker build --force-rm -t "${IMAGE_TAG_HYSTRIX_DASHBOARD}" "${WORKSPACE}/spring-petclinic-hystrix-dashboard"
docker build --force-rm -t "${IMAGE_TAG_VETS_SERVICE}" "${WORKSPACE}/spring-petclinic-vets-service"
docker build --force-rm -t "${IMAGE_TAG_VISITS_SERVICE}" "${WORKSPACE}/spring-petclinic-visits-service"
docker build --force-rm -t "${IMAGE_TAG_GRAFANA_SERVICE}" "${WORKSPACE}/docker/grafana"
docker build --force-rm -t "${IMAGE_TAG_PROMETHEUS_SERVICE}" "${WORKSPACE}/docker/prometheus"
```
    • Imagelari push etmek icin script yazdim. jenkins/push-staging-docker-images-to-ecr.sh
```bash
aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
docker push "${IMAGE_TAG_ADMIN_SERVER}"
docker push "${IMAGE_TAG_API_GATEWAY}"
docker push "${IMAGE_TAG_CONFIG_SERVER}"
docker push "${IMAGE_TAG_CUSTOMERS_SERVICE}"
docker push "${IMAGE_TAG_DISCOVERY_SERVER}"
docker push "${IMAGE_TAG_HYSTRIX_DASHBOARD}"
docker push "${IMAGE_TAG_VETS_SERVICE}"
docker push "${IMAGE_TAG_VISITS_SERVICE}"
docker push "${IMAGE_TAG_GRAFANA_SERVICE}"
docker push "${IMAGE_TAG_PROMETHEUS_SERVICE}"
```
    • Jenkins servera Rancher CLI yukledim.

```bash
curl -SsL "https://github.com/rancher/cli/releases/download/v2.4.9/rancher-linux-amd64-v2.4.9.tar.gz" -o "rancher-cli.tar.gz"
tar -zxvf rancher-cli.tar.gz
sudo mv ./rancher-v2.4.9/rancher /usr/local/bin/rancher
chmod +x /usr/local/bin/rancher
rancher --version
```

    • Rancher serverda API-KEY kurdum bu API-KEY vasitasiyla jenkins serverin rancherla gorusmesini sagladim. Rancher>sag ustteki kosedki isaret>API&Keys>Add key>Description>petclinic-key>Create bunu kaydettim cunku bir kez cikiyor. Jenkins>Dashboard>managejenkins>manage credentials>Jenkins global>global credentials>add credentials>username=rancherdaki cikan ekrandaki access keyi yapistirdim>password=secret key>id=rancher-petclinic-credentials(jenkinsfile da degisken olarak bu ismi kullanmistim)
    • 4. pipeline kurmak icin petclinic-staging adinda bir pipeline kurdum. jenkins/jenkinsfile-petclinic-staging. Project/namespaces>petclinic-staging-ns>sag uc nokta>view in api>project id> tirnak arasini kopyala>RANCHER_CONTEXT degiskenine kopyala.

```groovy
pipeline {
    agent { label "master" }
    environment {
        PATH=sh(script:"echo $PATH:/usr/local/bin", returnStdout:true).trim()
        APP_NAME="petclinic"
        APP_REPO_NAME="sezgin-repo/petclinic-app-staging"
        AWS_ACCOUNT_ID=sh(script:'export PATH="$PATH:/usr/local/bin" && aws sts get-caller-identity --query Account --output text', returnStdout:true).trim()
        AWS_REGION="us-east-1"
        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        RANCHER_URL="https://rancher.drsezginerdem.com"
        // Get the project-id from Rancher UI
        RANCHER_CONTEXT="petclinic-cluster:project-id" 
        RANCHER_CREDS=credentials('rancher-petclinic-credentials')
    }
//mavenla packaging
    stages {
        stage('Package Application') {
            steps {
                echo 'Packaging the app into jars with maven'
                sh ". ./jenkins/package-with-maven-container.sh"
            }
        }
//imagelara tag atadim
        stage('Prepare Tags for Staging Docker Images') {
            steps {
                echo 'Preparing Tags for Staging Docker Images'
                script {
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    env.IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
                    env.IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
                }
            }
        }
//imagelari build ettim
        stage('Build App Staging Docker Images') {
            steps {
                echo 'Building App Staging Images'
                sh ". ./jenkins/build-staging-docker-images-for-ecr.sh"
                sh 'docker image ls'
            }
        }
//imagelari push ettim
        stage('Push Images to ECR Repo') {
            steps {
                echo "Pushing ${APP_NAME} App Images to ECR Repo"
                sh ". ./jenkins/push-staging-docker-images-to-ecr.sh"
            }
        }
//imagelari deploy ettim
        stage('Deploy App on Petclinic Kubernetes Cluster'){
            steps {
                echo 'Deploying App on K8s Cluster'
                sh "rancher login $RANCHER_URL --context $RANCHER_CONTEXT --token $RANCHER_CREDS_USR:$RANCHER_CREDS_PSW" //rancher servera login olmak icin
                sh "envsubst < k8s/base/kustomization-template.yml > k8s/base/kustomization.yml" //env variable lari guncelliyor
                sh "rancher kubectl delete secret regcred -n petclinic-staging-ns || true" //rancherda kubectl yapmamizi saglayan komut. Olusturdugum namespacede regcred adinda bir secretim varsa onu sildim ki yeni olustururken sorun olmasin eger yoksa || true diyorum hata vermeden gec.
//asagidaki komutla ecr a girmek icin secret olusturdum ki ben de imagelarimi cekebileyim.
                sh """
                rancher kubectl create secret generic regcred -n petclinic-staging-ns 
                --from-file=.dockerconfigjson=$JENKINS_HOME/.docker/config.json \
                --type=kubernetes.io/dockerconfigjson
                """
                sh "rancher kubectl apply -k k8s/staging/" //bulundugu klasor icindeki customazation.yaml fileina bakacak
            }
        }
    }
//tum imagelari sil
    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
        }
    }
}
```

    • staging/replica-count.yaml fileinin icinde>host: larin isimlerini petclinic-staging-drsezginerdem.com olarak degistirdim.
    • commit ve push
```bash
git add .
git commit -m 'added jenkinsfile petclinic-staging for release branch'
git push --set-upstream origin feature/msp-26
git checkout release
git merge feature/msp-26
git push origin release
```
    • Jenkins>Dashboard>newitem>pipeline>petclinic-prod>OK>Build periodically> 59 23 * * 0> Pipeline script from SCM>Git>Repo URL>Branch>release>jenkins/jenkinsfile-petclinic-staging>SAVE and BUILD.
    • Kubernetesde Route 53 kurdum. Clusterdaki bir instancein publicIp adresini al>Route 53>hosted zone>domainName>create record>simple record>define simple record>petclinic-staging.drsezginerdem.com> Ip adress or another value>altina yapistir>define single record>create record. Bu islemlerle route 53 uzerinden siteme ulasabiliyorum.

# 27. MSP 27 - Prepare a Production Pipeline

Production Deployment Setup
Prepare a Production Pipeline
MSP-27
Prepare a Production Pipeline on Jenkins Server
feature/msp-27

    • Son pipelineimi yapiyorum. Masterda yani production ortaminda projemi cikariyorum. Herkes kullanacak. Bu pipelineda mastera bir degisikligi commit ettigim zaman otomatikman guncellesin webhookla. 
    • Feature ekledim.

git checkout release
git branch feature/msp-27
git checkout feature/msp-27

  • Production pipeline icin ecr repo olusturdum. jenkins Dashboard>new item>freestyleProject>create-ecr-docker-registry-for-petclinic-prod>OK>execute shell>
```bash   
      PATH="$PATH:/usr/local/bin"
      APP_REPO_NAME="sezgin-repo/petclinic-app-staging"
      AWS_REGION="us-east-1"
      
      aws ecr create-repository \
        --repository-name ${APP_REPO_NAME} \
        --image-scanning-configuration scanOnPush=false \
        --image-tag-mutability MUTABLE \
        --region ${AWS_REGION}
```      
    • BUILD. ECR olustu.
    • Imagelari taglemek icin script yazdim Jenkins/prepare-tags-ecr-for-prod-docker-images.sh
```bash
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
export IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
export IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
```

    • Build icin script yazdim jenkins/build-prod-docker-images-for-ecr.sh

```bash
docker build --force-rm -t "${IMAGE_TAG_ADMIN_SERVER}" "${WORKSPACE}/spring-petclinic-admin-server"
docker build --force-rm -t "${IMAGE_TAG_API_GATEWAY}" "${WORKSPACE}/spring-petclinic-api-gateway"
docker build --force-rm -t "${IMAGE_TAG_CONFIG_SERVER}" "${WORKSPACE}/spring-petclinic-config-server"
docker build --force-rm -t "${IMAGE_TAG_CUSTOMERS_SERVICE}" "${WORKSPACE}/spring-petclinic-customers-service"
docker build --force-rm -t "${IMAGE_TAG_DISCOVERY_SERVER}" "${WORKSPACE}/spring-petclinic-discovery-server"
docker build --force-rm -t "${IMAGE_TAG_HYSTRIX_DASHBOARD}" "${WORKSPACE}/spring-petclinic-hystrix-dashboard"
docker build --force-rm -t "${IMAGE_TAG_VETS_SERVICE}" "${WORKSPACE}/spring-petclinic-vets-service"
docker build --force-rm -t "${IMAGE_TAG_VISITS_SERVICE}" "${WORKSPACE}/spring-petclinic-visits-service"
docker build --force-rm -t "${IMAGE_TAG_GRAFANA_SERVICE}" "${WORKSPACE}/docker/grafana"
docker build --force-rm -t "${IMAGE_TAG_PROMETHEUS_SERVICE}" "${WORKSPACE}/docker/prometheus"
```
    • Imagelari push etmek icin script yazdim jenkins/push-prod-docker-images-to-ecr.sh

```bash
aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
docker push "${IMAGE_TAG_ADMIN_SERVER}"
docker push "${IMAGE_TAG_API_GATEWAY}"
docker push "${IMAGE_TAG_CONFIG_SERVER}"
docker push "${IMAGE_TAG_CUSTOMERS_SERVICE}"
docker push "${IMAGE_TAG_DISCOVERY_SERVER}"
docker push "${IMAGE_TAG_HYSTRIX_DASHBOARD}"
docker push "${IMAGE_TAG_VETS_SERVICE}"
docker push "${IMAGE_TAG_VISITS_SERVICE}"
docker push "${IMAGE_TAG_GRAFANA_SERVICE}"
docker push "${IMAGE_TAG_PROMETHEUS_SERVICE}"
```
    • pipeline icin jenkinsfile olusturdum. jenkins/jenkinsfile-petclinic-prod
```groovy
pipeline {
    agent { label "master" }
    environment {
        PATH=sh(script:"echo $PATH:/usr/local/bin", returnStdout:true).trim()
        APP_NAME="petclinic"
        APP_REPO_NAME="sezgin-repo/petclinic-app-prod"
        AWS_ACCOUNT_ID=sh(script:'export PATH="$PATH:/usr/local/bin" && aws sts get-caller-identity --query Account --output text', returnStdout:true).trim()
        AWS_REGION="us-east-1"
        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        RANCHER_URL="https://rancher.drsezginerdem"
        // Get the project-id from Rancher UI
        //yeni bir ns yaratip view api den api numarasini yapistirdim
        RANCHER_CONTEXT="c-rclgv:p-z8lsg"
        RANCHER_CREDS=credentials('rancher-petclinic-credentials')
    }
    stages {
        stage('Package Application') {
            steps {
                echo 'Packaging the app into jars with maven'
                sh ". ./jenkins/package-with-maven-container.sh"
            }
        }
        stage('Prepare Tags for Production Docker Images') {
            steps {
                echo 'Preparing Tags for Production Docker Images'
                script {
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    env.IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
                    env.IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
                }
            }
        }
        stage('Build App Production Docker Images') {
            steps {
                echo 'Building App Production Images'
                sh ". ./jenkins/build-prod-docker-images-for-ecr.sh"
                sh 'docker image ls'
            }
        }
        stage('Push Images to ECR Repo') {
            steps {
                echo "Pushing ${APP_NAME} App Images to ECR Repo"
                sh ". ./jenkins/push-prod-docker-images-to-ecr.sh"
            }
        }
        stage('Deploy App on Petclinic Kubernetes Cluster'){
            steps {
                echo 'Deploying App on K8s Cluster'
                sh "rancher login $RANCHER_URL --context $RANCHER_CONTEXT --token $RANCHER_CREDS_USR:$RANCHER_CREDS_PSW"
                sh "envsubst < k8s/base/kustomization-template.yml > k8s/base/kustomization.yml"
                sh "rancher kubectl delete secret regcred -n petclinic-prod-ns || true"
                sh """
                rancher kubectl create secret generic regcred -n petclinic-prod-ns \
                --from-file=.dockerconfigjson=$JENKINS_HOME/.docker/config.json \
                --type=kubernetes.io/dockerconfigjson
                """
                sh "rancher kubectl apply -k k8s/prod/"
            }
        }
    }
    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
        }
    }
}
```
      
    • 5. pipeline kurmak icin petclinic-staging adinda bir pipeline kurdum. jenkins/jenkinsfile-petclinic-prod. Github hook trigger>pipeline script>git>git URL>branch>*/master>jenkins/jenkins-petclinic-prod

    • Commit ve push.
git add .
git commit -m 'added jenkinsfile petclinic-production for master branch'
git push --set-upstream origin feature/msp-27
git checkout release
git merge feature/msp-27
git push origin release


    • relasei mastera merge ettim.

git checkout master
git merge release
git push origin master

    • BUILD.

# 28. MSP 28 - Setting Domain Name and TLS for Production Pipeline with Route 53

Production Deployment Setup
Set Domain Name and TLS for Production
MSP-28
Set Domain Name and TLS for Production Pipeline with Route 53
feature/msp-28

    • DNS record olusturdum.
    • Route 53>hosted zones>domaini sec>create record>simple routing>value>herhangi bir instance in ip numarasi>record name>sezginpetclinic.drsezginerdem.com(prod yaml dosyalarindaki adresinde degismis oldugunu kontrol ettim).
    • Su an http uzerinden baglaniyorum ancak https yani sitem icin sertifika almam gerekiyor. Daha once bunu zaten aws den yapmistim. Simdi haricten sertifika nasil alinir onu yapicam.
    • Kubernetes icin certifika manageri olan cert-manager toolunu clusterima yukledim (bu islemin hangi enviromenta nasil yapilacagi cert-managerin sayfasinda var), bu jenkins uzerinden ranchera kurup rancherda ayarlarini yapacagim. 
    • Rancher>global>sezgni-petclinic-prod>kubeconfig file>copy to clibboard. Jenkins ec2>nano petclinic-config> rancherdan aldigim dosyami buraya yapistirdim>ciktim>chmod 400 petclinic-config.
    • Kubectl komutlarini calistirirken daha once rancher server uzerinden islem yapiyordum ama ben artik petclinic uzerinden islem yapmak istiyorum dolayisiyla kubeconfig dosyasini degistirmem lazim bunun icin yeni bir secret dosyasi olusturdum. Bunun icinde credentialslarim var benim clusterima baglanmami sagliyor. Bunu kube dosyasina tanimlamam lazim. 

export KUBECONFIG=petclinic-config

    • bundan sonra yapacagim islemler petclinic clusterini etkileyecek. 
    • Cert-manageri yukledim.
```bash
kubectl create namespace cert-manager #ns create ettim
helm repo add jetstack https://charts.jetstack.io #helm repodan instal ettim
helm repo update #update ettim
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.1.0/cert-manager.crds.yaml
      helm install \
      cert-manager jetstack/cert-manager \
      --namespace cert-manager \
      --version v1.0.4
```
    • Ranchera certmanager agentini yukledim.
```bash
kubectl get pods --namespace cert-manager -o wide #her bir makina icin bir agent kurdum
```
    • Agentin lets encrypt ile irtibata gecip check yapmasi icin k8s/ClusterIssuer/tls-cluster-issuer-prod.yml (dokumantasyonunda var yalnizca mail adresini degistirdim)
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
  namespace: cert-manager
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    preferredChain: "ISRG Root X1"
    # Email address used for ACME registration
    email: drsezginerdem@gmail.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: nginx
```
    • bu yaml dosyasini calistiriyorum. Ve certificate managerimi yukledim/
```bash
export KUBECONFIG="$HOME/petclinic-config"
kubectl apply -f k8s/tls-cluster-issuer-prod.yml
kubectl get clusterissuers letsencrypt-prod -n cert-manager -o wide
```
    • Lets encript firmasi ile challenge yapip sertifika talebinde bulunacagim. Rancerdaki apigateway servisinin icine asagidaki yaml dosyasini ekleyecegim. rancher>sezgin-petclinic-prod>default>load balancer altindaki>api-gateway>view/edit YAML>annotations satirinin altina>
```yaml
annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
```
    • ekle. En ustteki Spec altina da>
```yaml
spec:
  tls:
  - hosts:
    - petclinic.clarusway.us
    secretName: petclinic-tls
```
    • Certifica basari ile alindi.
    • HTTPS ile baglantim basari ile gerceklesti.
    • Commit ve push

git add .
git commit -m 'added tls scripts for petclinic-production'
git push --set-upstream origin feature/msp-28
git checkout master
git merge feature/msp-28
git push origin master

# 29. MSP 29 - Monitoring with Prometheus and Grafana

Production Deployment Setup
Set Monitoring Tools
MSP-29
Set Monitoring tools, Prometheus and Grafana

 
    • Grafana ve prometheus 9090 I monitor etmesi gerekiyor. Onu duzeltmem gerekiyor. Rancher a geldim. Cluster namespace>resource>workload>service discovery>prometheus server>edit>show advanced options>portname 9090>publish the service port>9090. Yaml dosyalarindan yapacagim seyi rancher uzerinden yaptim daha anlasilir oluyor.
    • Pormetheus serverimi disardan erisilebilir hale getirmem icin nodeport olustirmam lazim. Service discovery>Prometheus-server in yaninda selector kisminin yanindaki target alanini kopyaladim>add record>name>prometheus-prod>namespace>petclinic-prod-ns>the set of pods which match a selector>key kismina az once kopyaladigim alani yapistirdim key ve value olarak ayirdi>show advanced options>nodeport/on every node>port name=tcp>publish the service port=9090>CREATE
    • rancherdaki prometheus servera tikladigimda prometheus aciliyor.
    • Grafana server icin bir record olusturdum. grafana-server>add record>grafana-prod>namespace>default>the set of pods which>add selector>label ve value kismina onceki kopyaladigimi yapistirdim sadece prometheus olan yere garafana yazdim>show advanced options>nodeport>add port>portname=tcp>publish service-port=3000>CREATE
    • Grafana serverima girdigimde gorebiliyorum. Grafana>dashboard altinda onceden hazirlanmis dashboard dosyasi var yaml halinde. 
