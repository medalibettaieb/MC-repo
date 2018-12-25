pipeline {
    agent {
dockerfile true	}

    stages {
        stage('Exemple') {
            steps {
                echo 'hello'
            }
        }
       

stage('Building image') {
      steps{
        script {
          docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
        }



    }

