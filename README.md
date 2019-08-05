# vmware_jenkins_nexus_pipeline

# Recommended Setup:

- A containerized jenkins instance
- A containerized nexus instance (v3).

*NOTE: if your jenkins and nexus instances are not containerized, the guidelines set out here are still applicable.*

## 1. Install plugins

1. In the jenkins master ui, install *Nexus Artifact Uploader*, *Pipeline Utility Steps* and *Nexus Platform Plugin*,

## 2. Login to nexus

2. Login to nexus in the browser. If username/password admin/admin123 does not work, ssh into your nexus container and cat the nexus password at /nexus-data/admin.password,

## 3. Create hosted nexus repository

3. Create a hosted repository on your nexus user in the browser ui, name it `files-scm` for now.

## 4. Create credentials in jenkins ui

4. In the jenkins ui, create credentials for your nexus user. Set the username and password to be the same as the username and password for when you login to nexus,

## 5. Add maven to your jenkins user

5. In the jenkins ui, navigate to Manage Jenkins > Global Tools Configuration and add Maven, in the name field enter "M3" for now,

## 6. Inlude pom.xml in GitHub repository

6. In the git repository that you want to upload, include a pom.xml file that contains the following content (for correct indentation see: https://github.com/ilyashusain/nginx_config/blob/master/pom.xml):

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.sonatype.training.nxs301</groupId>
  <artifactId>maven-deploy-example</artifactId>
  <version>1</version>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <distributionManagement>
    <repository>
      <id>nexus</id>
      <url>http://35.232.111.251:8081/nexus/content/repositories/releases</url>
    </repository>
    <snapshotRepository>
      <id>nexus</id>
      <url>http://35.232.111.251:8081/nexus/content/repositories/snapshots</url>
    </snapshotRepository>
  </distributionManagement>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```
