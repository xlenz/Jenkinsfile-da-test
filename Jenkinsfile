pipeline {
  agent { label 'master' }

  options { timeout(time: 1, unit: 'MINUTES') }

  parameters {
    string(name: 'siteName', defaultValue: 'TEST-site', description: 'Profile Name')
    
    string(name: 'component', defaultValue: 'TEST-site', description: 'Profile Name')
    string(name: 'fileIncludePatterns', defaultValue: '*.txt')
    string(name: 'fileExcludePatterns', defaultValue: '')
    
    string(name: 'deployApp', defaultValue: 'TEST-app', description: 'Application Name')
    string(name: 'deployEnv', defaultValue: 'TEST-env', description: 'Environment Name')
    string(name: 'deployProc', defaultValue: 'TEST-app-proc', description: 'Application Process Name')
        
    string(name: 'processName', defaultValue: 'TEST-GlobalProcess', description: 'Global Process Name')
    string(name: 'resourceName', defaultValue: 'TEST-RESOURCE', description: 'Resource Name')
    
    booleanParam(name: 'skip', defaultValue: true, description: 'Debug?')
  }

  stages {
    
    stage('Spawn artifacts') {
      steps {
        // timestamp plugin usage
        timestamps {
          echo "__Workspace ${env.WORKSPACE}"
          println "__Build number ${env.BUILD_NUMBER}"
        }
        
        sh 'env > env_variables.txt'      
      }
    }
    
    stage('DA Plugin') {
      steps {
        // Publish artifacts and run Application Process
        step([$class: 'SerenaDAPublisher',
          skip: params.skip,
          siteName: params.siteName,

          baseDir: env.WORKSPACE,
          fileIncludePatterns: params.fileIncludePatterns,
          fileExcludePatterns: params.fileExcludePatterns,
          component: params.component,
          versionName: env.BUILD_NUMBER,

          deploy: true,
          deployApp: params.deployApp,
          deployEnv: params.deployEnv,
          deployProc: params.deployProc
        ])
      }
    }
    
    stage('ui-tests') {
      steps {
        script {
          def browsers = ['chrome', 'firefox']
          for (int i = 0; i < browsers.size(); ++i) {
            echo "Testing the ${browsers[i]} browser"
            sleep 1
          }
        }
      }
    }
    
  }

  post {
    always {
      // artifacts archiver plugin usage
      step([$class: 'ArtifactArchiver', artifacts: '*.txt'])
    }
  }
  
}
