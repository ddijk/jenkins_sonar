#!/usr/bin/env groovy

pipeline {
    agent any
    environment {
        nexusCredentialsId = 'jenkins-nexus'
        pom = readMavenPom file: 'pom.xml'
    }

    tools {
        maven 'maven_3.5.4'
        jdk 'OpenJDK10'
    }

    stages {
         // to prevent endless loop, abort when Jenkins job is triggerd by Maven Release commits
         stage('Check for GitLab trigger on commit by Maven Release Plugin') {
             steps {
                 script {
                     checkout scm
                     if (lastCommitIsBumpCommit()) {
                         currentBuild.result = 'ABORTED'
                         error('Last commit bumped the version, aborting the build to prevent a loop.')
                     } else {
                         echo('Last commit is not a bump commit, job continues as normal.')
                     }
                 }
             }
        }
        stage('Build, Test & SonarQube Scan') {
                  steps {
                       echo 'expected branch = ' + env.BRANCH_NAME
                       withSonarQubeEnv('StuComm SonarQube Server') {
                         sh 'mvn clean test sonar:sonar -P sonar -Dsonar.branch=' + env.BRANCH_NAME
                        }
                  }
        }

        stage("Quality Gate") {
                  steps {
                     timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                       script {
                       def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                       if (qg.status != 'OK') {
                         error "Pipeline aborted due to quality gate failure: ${qg.status}"
                       }
                       }
                     }
                  }
        }
        stage('Package & upload to Nexus') {
            when {
                branch "develop"
            }
            steps {
                echo 'deploying ' + env.BRANCH_NAME
                sh 'mvn release:prepare release:perform -Dmaven.test.skip=true'
            }
        }
    }
}
private boolean lastCommitIsBumpCommit() {
    lastCommit = sh([script: 'git log -1', returnStdout: true])
    return lastCommit.contains("[maven-release-plugin]")
}
