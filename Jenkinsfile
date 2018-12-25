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
          docker.build  ":$BUILD_NUMBER"
        }
      }
    }
        }



    }

