# vmware_jenkins_nexus_pipeline

# Recommended Setup:

- A containerized jenkins instance
- A containerized nexus instance (v3).

*NOTE: if your jenkins and nexus instances are not containerized, the guidelines set out here are still applicable.*

## 1. Install plugins

In the jenkins master ui, install *Nexus Artifact Uploader*, *Pipeline Utility Steps* and *Nexus Platform Plugin*,

## 2. Login to nexus

Login to nexus in the browser. If username/password admin/admin123 does not work, ssh into your nexus container and cat the nexus password at /nexus-data/admin.password,

## 3. Create hosted nexus repository

Create a hosted repository on your nexus user in the browser ui, name it `files-scm` for now.

## 4. Create credentials in jenkins ui

In the jenkins ui, create credentials for your nexus user. Set the username and password to be the same as the username and password for when you login to nexus,

## 5. Add maven to your jenkins user

In the jenkins ui, navigate to Manage Jenkins > Global Tools Configuration and add Maven, in the name field enter "M3" for now,

## 6. Inlude pom.xml in GitHub repository

In the git repository that you want to upload, include a pom.xml file that contains the following content (for correct indentation see: https://github.com/ilyashusain/nginx_config/blob/master/pom.xml):

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

## 7. Create pipeline

Create a Jenkins pipeline in the web ui and copy this into the pipeline script:

```
pipeline {
    agent {
        label "master"
    }
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "M3"
    }
    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "35.232.111.251:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "files-scm2"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
    }
    stages {
        stage("clone code") {
            steps {
                script {
                    // Let's clone the source
                    git 'https://github.com/ilyashusain/nginx_config.git';
                }
            }
        }
        stage("mvn build") {
            steps {
                script {
                    // If you are using Windows then you should use "bat" step
                    // Since unit testing is out of the scope we skip them
                    sh "mvn package -DskipTests=true"
                }
            }
        }
        stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
    }
}
```
