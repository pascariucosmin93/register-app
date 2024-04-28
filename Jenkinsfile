pipeline {
  agent {
    label 'Linux'
  }
  stages {
    stage('Log Tool Version') {
      steps {
        sh '''
          mvn --version
          git --version
          java -version
        '''
      }
    }

    stage('Check for POM') {
      steps {
        sh 'test -f pom.xml'
      }
    }

    stage('Build with Maven') {
      steps {
        sh 'mvn compile'
      }
    }

    stage('Run Tests') {
      steps {
        sh 'mvn test'
      }
    }

    stage('Run Static Code Analysis') {
      steps {
        build job: 'static-code-analysis'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'sudo docker build -t cameronmcnz/cams-rps-service .'
      }
    }

    stage('Push Docker Image') {
      steps {
        sh 'sudo docker push cameronmcnz90210/cams-rps-service:first'
      }
    }

    stage('Create Executable JAR File') {
      steps {
        sh 'mvn package spring-boot:repackage'
      }
    }

    stage('Deploy to AWS') {
      steps {
        script {
          def response = input message: 'Should we push to DockerHub?',
            parameters: [choice(choices: 'Yes\nNo',
              description: 'Proceed or Abort?',
              name: 'What to do???')]

          if (response == "Yes") {
            sh 'aws ecs update-service --cluster rps-cluster --service rps-service --force-new-deployment'
          }
          if (response == "No") {
            writeFile(file: 'deployment.txt', text: 'We did not deploy.')
          }
        }
      }
    }
  }
}
