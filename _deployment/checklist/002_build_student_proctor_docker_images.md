---
title: Build Student/Proctor Docker Images
permalink: "/deployment/checklist/build_student_proctor_docker_images.html"
layout: "document"
categories: ["deployment", "checklist", "tds"]
---
# Student and Proctor Images
### Student

1. Clone (git) Student master branch: `git clone https://github.com/SmarterApp/TDS_Student/tree/master`
1. Place the items and stimuli that you want to use in this deployment in on the machine you’re using to build the project: `/usr/local/tomcat/resources/tds/bank`
  - Practice and training tests can be downloaded from FTP site: `ftp://ftps.smarterbalanced.org/~sbacpublic/Public`
1. Run `mvn clean install -DskipTests` in the cloned project’s base directory. 
1. Change directory to the student directory `cd student`
1. Run `mvn docker:build` 
1. Tag the docker image that was just built with the Student EC2 Repository you created earlier.
  * Run `docker images` to find the IMAGE ID for what was just built.
  * Run `docker tag <IMAGE ID> <STUDENT EC2 Repository>:latest`
  * Example: 
  `docker tag 7b6edb762795 269146879732.dkr.ecr.us-west-2.amazonaws.com/student:latest`
1. Run `eval $(aws ecr get-login)`
1. Run `docker push <STUDENT EC2 Repository>:latest`

### Proctor

1. Clone (git) Proctor master branch : `git clone https://github.com/SmarterApp/TDS_Proctor/tree/master`
1. This should be done if you already built student.  Place the items and stimuli that you want to use in this deployment on the machine you’re using to build this project in `/usr/local/tomcat/resources/tds/bank`
1. Change directory to the proctor directory (cd proctor)
1. Run `mvn clean install -DskipTests` in the cloned project’s proctor directory. 
1. Get yourself a Proctor SAML xml metadata file and associated keystore *.jks file.
  * Notes for this can be found [http://www.smarterapp.org/deployment/checklist/art.html](http://www.smarterapp.org/deployment/checklist/art.html)
1. Put the metadata file and keystore in `/opt/sbtds/deploy/resources/security`
1. Run `mvn docker:build `
1. Tag the docker image that was just built with the Proctor EC2 Repository you created earlier.  
    - Run `docker images` to find the `IMAGE ID` for what was just built.
    - Run `docker tag <IMAGE ID> <PROCTOR EC2 Repository>:latest`
    - Example: `docker tag 7b6edb762795 269146879732.dkr.ecr.us-west-2.amazonaws.com/proctor:latest`

1. Run `eval $(aws ecr get-login)`
1. Run `docker push <PROCTOR EC2 Repository>:latest`

[back to Deployment Checklists](index.html)

