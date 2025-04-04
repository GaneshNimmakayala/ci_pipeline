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
                        myImage = docker.build registry
                        myImage.push()
                    }
                }
            }
        }
         stages {
        stage('Run Trivy Scan') {
            steps {
                script {
                    // Run Trivy scan and capture the JSON output
                    def trivyScanResult = sh(script: 'trivy --exit-code 1 --severity HIGH,CRITICAL --quiet --format json /root/workspace/Jenkins_Project/target', returnStdout: true).trim()

                    // Parse the JSON output
                    def jsonResult = readJSON text: trivyScanResult
                    
                    // Check if there are any HIGH or CRITICAL vulnerabilities
                    def highSeverityFound = jsonResult.find { vuln -> vuln.Vulnerabilities.find { it.Severity == 'HIGH' || it.Severity == 'CRITICAL' } }
                    
                    if (highSeverityFound) {
                        error "High or critical vulnerabilities found in the scan. Failing the build."
                    } else {
                        echo "No high or critical vulnerabilities found."
                    }
                }
            }
        }
    }
    }
}
                 


