---
title: Deployment Pre-Requisites
permalink: "/deployment/checklist/student_3x/general_tools.html"
layout: "document"
categories: ["deployment", "checklist", "tds", "3x"]
---

# General Tools

The following tools are required to deploy the application.

### Kops

The Kubernetes Operations tool (kops) is used for interacting with the Kubernetes cluster that will host the TDS environment.  Details for installation can be found [here](https://github.com/kubernetes/kops) and a tutorial for use can be found [here](https://github.com/kubernetes/kops/blob/master/docs/aws.md).  **_It is important to be on the latest version_**.

### Kubectl

The kubectl command line tool is used for deploying and managing applications within the Kubernetes cluster.  This program is used to inspect cluster resources, create/update/delete components, etc.  Installation instructions can be found [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

### Docker

[Docker](https://www.docker.com/) is required for building and maintaining the container images.  Additionally, docker is required for pushing to/pulling from the Amazon ECR and/or DockerHub repositories.  Installation steps can be found [here](https://store.docker.com/search?offering=community&q=&type=edition).

### AWS command line tools

The AWS Command Line Interface (aws-cli) is required for interacting with AWS (particularly ECR).  Additionally, the aws-cli is a prerequisite for kops and kubectl.  More information and how to install the command line tools can be found [here](https://aws.amazon.com/cli/).  Once installed you should be able to connect to your AWS EC2 Container Repository (ECR).  Run aws ecr describe-repositories and ensure you get results.

### Maven

Maven is required to build the Student and Proctor images with embedded items and stimuli. 

- [Java 7](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html)
- [Java 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
- [Maven](https://maven.apache.org/install.html):  Apache Maven is required for building the Student and Proctor application .war files as well as their docker container images.
    


The toolchains.xml file is used by maven to determine the path to the correct JDK for the specified Java version.  The toolchains.xml file should be saved in Maven’s repository directory; the default location is ~/.m2.  Update the jdkHome values (highlighted in yellow) to use the path for the Java 7 and Java 8 JDKs.  An example toolchains.xml file is shown below:

 `toolchains.xml`
 
```xml
<?xml version="1.0" encoding="UTF8"?>
<toolchains>
  <!-- JDK toolchains -->
  <toolchain>
    <type>jdk</type>
    <provides>
      <version>1.7</version>
    </provides>
    <configuration>
      <jdkHome>/Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home</jdkHome>
    </configuration>
  </toolchain>
  <toolchain>
    <type>jdk</type>
    <provides>
      <version>1.8</version>
    </provides>
    <configuration>
      <jdkHome>/Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home</jdkHome>
    </configuration>
  </toolchain>
</toolchains>
```
The settings.xml file is used by maven to determine where dependencies should be downloaded from.  The settings.xml file should be saved in Maven’s repository directory; the default location is ~/.m2.  An example settings.xml file is shown below:

`settings.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<settings xsi:schemaLocation='http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd'
          xmlns='http://maven.apache.org/SETTINGS/1.0.0' xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance'>

    <profiles>
        <profile>
            <repositories>
                <repository>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                    <id>central</id>
                    <name>libs-releases</name>
                    <url>https://airdev.jfrog.io/airdev/libs-releases</url>
                </repository>
                <repository>
                    <snapshots />
                    <id>snapshots</id>
                    <name>libs-snapshots</name>
                    <url>https://airdev.jfrog.io/airdev/libs-snapshots</url>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                    <id>central</id>
                    <name>plugins-releases</name>
                    <url>https://airdev.jfrog.io/airdev/plugins-releases</url>
                </pluginRepository>
                <pluginRepository>
                    <snapshots />
                    <id>snapshots</id>
                    <name>plugins-snapshots</name>
                    <url>https://airdev.jfrog.io/airdev/plugins-snapshots</url>
                </pluginRepository>
            </pluginRepositories>
            <id>artifactory</id>
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>artifactory</activeProfile>
    </activeProfiles>
</settings>
```

### Spring CLI
Installing the Spring Boot Command-Line-Interface is required to encrypt sensitive information such as passwords before committing them to the GIT configuration repository.  This allows providing broader access to the configuration repository without exposing sensitive data.

- Instructions for installing the Spring Boot CLI can be found [here](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started-installing-spring-boot.html#getting-started-installing-the-cli).
- Instructions for installing the Spring Cloud CLI Extension can be found [here](https://cloud.spring.io/spring-cloud-cli/). 
- If required, download and install the “Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files” from Oracle.
- You can now encrypt/decrypt configuration values via:<br>
`spring encrypt <SENSITIVE_VALUE> --key <ENCRYPT_KEY>`<br>
`spring decrypt <ENCRYPTED_VALUE> --key <ENCRYPT_KEY>`
- Encrypted values can be used in configuration .yml files as:  
`password: '{cipher}encrypted_value'`

### Spring Cloud Configuration
Configuration is done using Spring Cloud.  Best practices recommend using a private Git repository to host these configuration files with a recommendation of using Gitlab since it provides free private hosting.  Only one person needs to do this step and any other deployer can simply clone and modify the data using standard git commands.

- Create a gitlab account (recommended because it offers free private repos)
- Create a gitlab repository to host the files located inside the configuration directory in the  tds_deployment_support_files.zip file. 

[back to Deployment Checklists](index.html)