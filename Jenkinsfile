pipeline {
    agent { label 'slave' }

    environment {    
        registry = "ganeshnimmakayala/jenkinsci"
        registry_cred = "docker_hub"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Performing SCM Checkout"
                git branch: 'master', url: 'https://github.com/GaneshNimmakayala/Banking_Project.git'       
            }
        }

        stage('Application Build') {
            steps {
                echo "Building Package"
                sh 'mvn clean package'
            }
        }

        stage('Code Coverage') {
            steps {
               echo "Running Code Coverage"
               sh "mvn jacoco:report"
            }
        }

        stage('SCA') {
            steps {
                echo "Running SCA"
                sh "mvn org.owasp:dependency-check-maven:check"
            }
        }

        stage('SAST') {
            steps {
                echo "Running SAST"
                withSonarQubeEnv('sonarqube_server1'){
                    sh 'mvn sonar:sonar -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html -Dsonar.projectName=Ganesh'
                }  
            }
        }

        stage('Quality Gates') {
            steps {
                echo "Running QualityGate in Sonarqube"
                script {
                   timeout(time:1, unit: 'MINUTES'){
                       def qg = waitForQualityGate()
                       if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                       }
                   }
                }
            }
        }

        stage('Build Image') {
            steps {
                echo "Building Docker Image"
                script {
                    docker.withRegistry( '', registry_cred ) { 
                        myImage = docker.build ${registry}
                        myImage.push()
                    }
                }
            }
        }

        stage('Scan Image') {
            steps {
                echo "Scanning Image"
                sh "trivy image --scanners vuln --offline-scan ganeshnimmakayala/jenkinsci:latest > trivyresults.txt"
            }
        }
    }
}

