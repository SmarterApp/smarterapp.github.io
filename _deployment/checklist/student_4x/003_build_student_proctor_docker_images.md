---
title: Build Proctor Docker Image
permalink: "/deployment/checklist/student_4x/build_student_proctor_docker_images.html"
layout: "document"
categories: ["deployment", "checklist", "tds", "4x"]
---
# Proctor Image
# TDS Environment Setup/Deployment
### DB setup
The assumption is that one has already followed the necessary steps to stand up a DB server covered in previous deployment checklist items.  However, there are some new schemas and tables that are required for the deployment.  Within the tds_deployment_support_files.zip file there is a db directory with two SQL scripts that can be used to create application tables.

1. Log into the TDS database
1. Run the `exam_create_schema.sql`
  - This will create the exam database with the appropriate tables
  - The user that you want the application to use should have read and write abilities on these tables.
1. Run the `exam_audit_create_schema.sql`
  - This will create the exam_audit database with the appropriate tables
  - The user that you want the application to use should have read and write abilities on these tables.
1. The user that you want the application should have the following DB access
  - Write access: exam, exam_audit, and session database
  - Read access: itembank, configs database

### EC2 Container Service
 Create [EC2 Container Repositories](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_Console_Repositories.html) to host Proctor Docker images (separate one for each) containing the SAML and security information.
### S3 Configuration for Items
 The item scoring functionality requires items and stimuli to be stored on S3 so that regardless of the number of exam microservices they all share the same item and stimuli. The following section lists the steps to create and set up an S3 bucket. This section does assume that you have an AWS account with the ability to create buckets on s3.
### Uploading Items and Stimuli to S3
Create a bucket that will house the items.  We also suggest creating a folder within that bucket to isolate the items and stimuli for that deployment.

For example, the bucket can be `tds-resources` with a folder `uat`
```
tds-resources
  uat
    items
      item-187-1234
 item-187-1234.xml
 metadata.xml
    stimuli
       stim-187-1234
  stim-187-1234.xml
  metadata.xml
```

### Build Proctor Docker image

1. Clone (git) Proctor master branch : `git clone https://github.com/SmarterApp/TDS_Proctor.git`
2. Ensure that you are using the proper tag.
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

