pipeline {
    agent {
        docker {
            image 'maven:3.9.4-eclipse-temurin-17-alpine' 
            args '-v /root/.m2:/root/.m2' 
        }
    }
    stages {
        stage('Linting') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage('Static Analysis') {
            steps {
                sh 'mvn pmd:pmd'
            }
        }
        stage('Build') { 
            steps {
                sh 'mvn -B -DskipTests clean package' 
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Deliver') {
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
        }
        stage('Copy to server') {
            steps {
                sh 'scp target/${NAME}-${VERSION}.jar user:${REMOTE_PASS}@remote.com/path/etc'
            }
        }
    }
    post {
        failure {
            emailext (
              subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
              body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
              recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }
        always {
            warningsNextGeneration([
                checkstyle(pattern: '**/target/checkstyle-result.xml'),
                pmd(pattern: '**/target/pmd.xml')

            ])
        }
    }
}

