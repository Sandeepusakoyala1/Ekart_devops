pipeline {
    agent any
    
    tools {
        // Define Maven and JDK tools
        maven 'maven3'
        jdk 'jdk17'
    }
    
    environment {
        // Set environment variable for SonarQube scanner home
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                // Checkout code from Git repository
                git branch: 'main', url: 'https://github.com/Sandeepusakoyala1/Ekart_devops.git'
            }
        }
        
        stage('Maven clean the repo') {
            steps {
                // Clean Maven project
                sh "mvn clean"
            }
        }
        
        stage('Compiling the source code') {
            steps {
                // Compile Maven project
                sh "mvn compile"
            }
        }
        
        stage('Unit test') {
            steps {
                // Run unit tests
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('SonarQube analysis') {
            steps {
                // Run SonarQube scanner
                withSonarQubeEnv('sonar-scanner') {
                    sh """${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=EKART \
                    -Dsonar.projectName=EKART \
                    -Dsonar.java.binaries=. """
                }
            }
        }
        
        stage('OWASP Dependency check') {
            steps {
                // Run OWASP Dependency-Check
                dependencycheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps {
                // Build the Maven project
                sh 'mvn package -DskipTests=true'
            }
        }
        
        stage('Deploy artifact to nexus') {
            steps {
                // Deploy artifact to Nexus
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }
        
        stage('Build and tag docker Image') {
            steps {
                // Corrected 'scripts' to 'steps'
                steps {
                    // This step should not normally be used in your script. Consult the inline help for details.
                    withDockerRegistry(credentialsId: 'dockercreds', url: 'https://hub.docker.com') {
                        sh "docker build -t sandeepusakoyalaa/ekart:latest -f docker/Dockerfile ."
                    }
                }
            }
        }
        
        stage('Trivy Scan') {
            steps {
                // Corrected 'sh' to 'steps'
                steps {
                    sh "trivy image sandeepusakoyalaa/ekart:latest > trivy-report.txt"
                }
            }
        }
        
        stage('Push the docker image to docker hub') {
            steps {
                // Corrected 'scripts' to 'steps'
                steps {
                    // This step should not normally be used in your script. Consult the inline help for details.
                    withDockerRegistry(credentialsId: 'dockercreds', url: 'https://hub.docker.com') {
                        sh "docker push sandeepusakoyalaa/ekart:latest"
                    }
                }
            }
        }
        
        stage('Kubernetes deploy') {
            steps {
                // Corrected 'withKubeConfig' parameter names and removed extra indentation
                withKubeConfig(credentialsId: 'K8-token', serverUrl: 'https://172.31.38.241:6443', namespace: 'webapps') {
                    sh "kubectl apply -f deploymentservice.yml -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}
