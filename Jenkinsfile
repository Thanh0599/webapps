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
        sh 'docker run gesellix/trufflehog --json https://github.com/LoveMeSensei/webapp.git > trufflehog'
        sh 'cat trufflehog'
            }
        }

        stage ('Source Composition Analysis') {
      steps {
         sh 'cd /home/New'
         sh 'git clone https://github.com/LoveMeSensei/webapp.git'
         sh 'cd /webapp'
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
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
                sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@54.234.61.152:/prod/apache-tomcat-8.5.78/webapps/webapp.war'
              }      
           }       
        }
    }
}
