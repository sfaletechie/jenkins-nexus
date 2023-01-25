pipeline {
    agent any
    tools {
        maven "Maven"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "192.168.29.42:8081"
        NEXUS_REPOSITORY = "java-app"
        NEXUS_CREDENTIAL_ID = "adminnexus"
    }
    stages {
        stage("Clone code from GitHub") {
            steps {
                script {
                    git branch: 'main', credentialsId: 'github_auth_sfale', url: 'https://github.com/sfaletechie/jenkins-nexus.git';
                }
            }
        }
        stage("TEST"){
            steps{
                sh 'mvn test'
                slackSend channel: 'devops-notifications', message: 'this is TEST stage $BUILD_ID'
            }
        }
        stage("Build"){
            steps{
                slackSend channel: 'devops-notifications', message: 'this is BUILD stage'
                sh 'mvn package'
            }
        }
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
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
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
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
        stage("deployOnTest"){
            steps{
                slackSend channel: 'devops-notifications', message: 'this is deployOnTest stage'
                deploy adapters: [tomcat7(credentialsId: 'tomcat8details', path: '', url: 'http://192.168.29.22:8080/')], contextPath: '/app1', onFailure: false, war: 'target/my-app-1.0-SNAPSHOT.jar'
            }
        }
        stage("deployOnPROD"){\
             input {
                message 'continueToPROD'
            }
            steps{
                slackSend channel: 'devops-notifications', message: 'this is deployOnPROD stage'
                deploy adapters: [tomcat7(credentialsId: 'tomcat8details', path: '', url: 'http://192.168.29.42:8081/')], contextPath: '/app1', onFailure: false, war: 'target/my-app-1.0-SNAPSHOT.jar'
            }
        }
    }
    post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}
