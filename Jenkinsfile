pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk23'
    }
    
    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
    }
    
    stages {
        stage('Maven Test') {
            steps {
                sh "mvn clean test"
            }
        }

        stage('Maven Install') {
            steps {
                sh "mvn clean install"
            }
        }

        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Petclinic-App \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Petclinic \
                        -Dsonar.jacoco.reportPaths=target/site/jacoco/jacoco.xml
                    '''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                script {
                    timeout(time: 60, unit: 'MINUTES') {
                        dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                        // dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                        dependencyCheckPublisher failedTotalCritical: 0, 
                                               failedTotalHigh: 0, 
                                               pattern: 'dependency-check-report.xml', 
                                               stopBuild: true, 
                                               unstableTotalCritical: 0, 
                                                unstableTotalHigh: 0
                    }
                }
            }
        }
    }
}
