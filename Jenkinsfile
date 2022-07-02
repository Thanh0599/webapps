pipeline {
    agent any
    tools {
        maven "Maven"
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    ''' 
            }
        }

        stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/Thanh0599/webapps.git > trufflehog'
        sh 'cat trufflehog'
            }
        }

        stage ('Source Composition Analysis') {
      steps {
         sh 'wget "https://raw.githubusercontent.com/Thanh0599/webapps/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
            }
        }

        stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
                }
            }
        }

        stage ('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@3.87.75.198:/prod/apache-tomcat-8.5.78/webapps/webapp.war'
              }      
           }       
        }

         stage ('DAST') {
      steps {
        sshagent(['tomcat']) {
                sh 'ssh -o  StrictHostKeyChecking=no ec2-user@107.23.252.86 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://3.87.75.198:8080/webapp/" || true'
                }
            }
        }
    }
}
