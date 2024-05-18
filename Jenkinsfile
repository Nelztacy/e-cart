pipeline {
    agent any

    tools {
        maven 'localMaven'
        jdk 'jdk 17'
    }

    environment {
        SCANNER_HOME = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
        NEXUS_CREDENTIAL_ID = 'NEXUS_CREDENTIAL_ID'
        WORKSPACE = "${env.WORKSPACE}"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Nelztacy/e-cart.git'
            }
        }

        stage('Mvn Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Mvn Unit Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sona-ecart') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=ECART \
                        -Dsonar.projectName=ECART \
                        -Dsonar.java.binaries=.
                    '''
                }
            }
        }

        stage('OWASP Dependencies Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Artifact Build') {
            steps {
                sh "mvn package -DskipTests=true"
                sh 'ls -l target'
            }
        }
        
        
        // stage('Artifact Build') {
        //     steps {
        //         sh "mvn package -DskipTests=true"
        //         sh 'ls -l target' // List the contents of the target directory to verify the artifact
        //     }
        //     post {
        //         success {
        //             echo 'Now Archiving'
        //             archiveArtifacts artifacts: '**/*.war'
        //         }
        //     }
        // }

        // stage('Nexus Artifact Uploader') {
        //     steps {
        //         script {
        //             def warFile = "${WORKSPACE}/target/webapp.war"
        //             if (fileExists(warFile)) {
        //                 nexusArtifactUploader(
        //                     nexusVersion: 'nexus3',
        //                     protocol: 'http',
        //                     nexusUrl: '10.0.0.116:8081',
        //                     groupId: 'webapp',
        //                     version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
        //                     repository: 'maven-project-releases',
        //                     credentialsId: "${NEXUS_CREDENTIAL_ID}",
        //                     artifacts: [
        //                         [artifactId: 'webapp',
        //                          classifier: '',
        //                          file: warFile,
        //                          type: 'war']
        //                     ]
        //                 )
        //             } else {
        //                 error "File ${warFile} does not exist"
        //             }
        //         }
        //     }
        // }
    }
}
