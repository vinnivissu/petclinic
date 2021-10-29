pipeline {
    agent { label 'main'}

    environment {
        MVN_GOAL='package'
    }

    tools {
        maven 'M3'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/akmaharshi/petclinic.git'
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean ${MVN_GOAL}"
            }
        }
        stage('Sonar Scan') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    sleep 20
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Post-Build Actions') {
            parallel {
                stage('Archive Artifacts') {
                    steps {
                        archiveArtifacts artifacts: "target/*.?ar", followSymlinks: false
                    }
                }
                stage('JUnit tests') {
                    steps {
                        junit "target/surefire-reports/*.xml"
                    }
                }
            }   
        }
    }

    post {
        success {
            notify('Success','good')
        }
        failure {
            notify('Failed','#FF0000')
        }
    }
}

def notify(status,colour) {
    emailext (
    to: 'devops.kphb@gmail.com',
    subject: "${status}: ${env.JOB_NAME}- ${env.BUILD_NUMBER}", 
    body: """<p>${status}: ${env.JOB_NAME}- ${env.BUILD_NUMBER}</p>
        <p>Check console output at <a href=${env.BUILD_URL}>${env.JOB_NAME}</a></p>
    """
    )
    
    slackSend color: "${colour}", message: "${status}: ${env.JOB_NAME}- ${env.BUILD_NUMBER} - URL: ${env.BUILD_URL}"
}
