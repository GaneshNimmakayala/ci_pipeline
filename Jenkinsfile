pipeline {
    agent any
    tools{
        maven 'maven'
    }

    environment {   
        registry = "ganeshnimmakayala/jenkinsci"
        registry_cred = "docker_hub"
    }
    parameters{
        choice(name: 'Choose version below', choices: [ '1', '2', '3' ], description: 'version of the code')
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
                withSonarQubeEnv('Sonarserver'){
                    sh 'mvn sonar:sonar -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html -Dsonar.projectName=Ganesh'
                }  
            }
        }

        stage('Quality Gates') {
            steps {
                echo "Running QualityGate in Sonarqube"
                script {
                   timeout(time:5, unit: 'MINUTES'){
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
                     def imageName = "ganeshnimmakayala/jenkinsci"
                     def imageTag = "${BUILD_NUMBER}"
                    docker.withRegistry( '', registry_cred ) { 
                        def myImage = docker.build("${imageName}:${imageTag}")
                        myImage.push()
                    }
                }
            }
        }
        stage('Run Trivy Scan') {
            steps {
                script {
                  echo "Scanning Docker Image"
                  sh "trivy image --scanners vuln --offline-scan ganeshnimmakayala/jenkinsci:latest > trivyresults.txt"
                    }
    
           }
         } 
        stage('Trigger cd-pipeline') {
            steps {
                build job: 'CD_PIPELINE'
            }
        }
    }
}
                 


