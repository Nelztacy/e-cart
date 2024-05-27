pipeline {
    agent any

    tools {
        maven 'localMaven'
        jdk 'jdk 17'
    }

    environment {
        SCANNER_HOME = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
         // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "10.0.0.116:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "maven-project-releases"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "NEXUS_CREDENTIAL_ID"
        ARTIFACT_VERSION = "${BUILD_NUMBER}"
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
                            version: ARTIFACT_VERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging]
                            ]
                        );

                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t nelzone/ecart:latest -f docker/Dockerfile ."
                    }
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image nelzone/ecart:latest > trivy-report.txt "
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker push nelzone/ecart:latest"
                    }
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                kubeconfig(caCertificate: '''-----BEGIN CERTIFICATE-----
MIIDBTCCAe2gAwIBAgIICKNt9xSsr3kwDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
AxMKa3ViZXJuZXRlczAeFw0yNDA0MDQxNDIzNTBaFw0zNDA0MDIxNDI4NTBaMBUx
EzARBgNVBAMTCmt1YmVybmV0ZXMwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQDI0MJ7PKnmvkYk0GZSdICfl3Hc5srVxX0n9pxO2e+IgiZpFgQ55bO6Q2Kz
ATJGqkFUMQNMTFQbrK997uLalt5hU+AhkXFqLctUa+ZFI4dxQsMQryOfz9B4+6gN
5ojQFb1NxQ+2lQ+fFd3bso+xxuoaLhe5T6FIR+0KW0DSYsaxrV5KU9WBC5glELEA
W4ELjUpMkjHcqRHLxZqvLks652pksJP3bTd6zv3MyGN8bzHE7aG53MdI7hYbUEsR
kKSuWCsLNL+Myf4GFAvI5R5Bs7UYaKZPfIIfC8Lci4BgeP/QFwtrhHbOIYSky/c1
tdoXTfMAR/0ZCIIbLVfxMrCkXT3FAgMBAAGjWTBXMA4GA1UdDwEB/wQEAwICpDAP
BgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBTvRhgVwPLPlY6jJvu32aU0panN0TAV
BgNVHREEDjAMggprdWJlcm5ldGVzMA0GCSqGSIb3DQEBCwUAA4IBAQC06Hqx4QNF
BpmcA6/pJLt88xJ69dHDH5Xdtzz9AlWtdIg2n3kyPZCyzyNHeeqG5VbU8voEU10h
fAqVoMwoA0MRDnPMfiDw/R9VdwPUWnGWjOO2JU/oDtTHJ1LlnWSRby/MHsmi6RkN
dwpO0BGHoH4JmGWe/pjrE7iqLsEiNdQWoO09C5Hg99xtWTvPiA2rtmNzir9MjD08
0k6kKYfstsLI0/bItw1BB74ZvFqH8E3j6V5Bm+kPHXd+y4WzxU1UIgqvIlULHM5j
fdcRwhZ6eJ/WBEdow5cD6Vo9yyxkzYHD40tB/YQVMNz8gHNAe6gndAyTkSItQ+qf
jQ27NGxgyI6Z
-----END CERTIFICATE-----''', credentialsId: 'Kubernetes-Cred', serverUrl: 'https://10.0.0.90:6443') {
    }
                    sh "kubectl apply -f deploymentservice.yml"
                    sh "kubectl get svc -n jenkins"
            }
        }
    }
}
}