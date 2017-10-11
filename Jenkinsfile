pipeline {
  agent { label 'master' }

  stages {
    stage('Example') {
      steps {
        timestamps {
          echo "__Workspace ${env.WORKSPACE}"
          println "__Build number ${env.BUILD_NUMBER}"
        }
        sh 'env > env_variables.txt'      
        sleep 3

        step([$class: 'ArtifactArchiver', artifacts: '*.txt'])
      }
    }
  }
}
