pipeline {
  agent { label 'master' }

  options { timeout(time: 1, unit: 'MINUTES') }

  parameters {
    string(name: 'siteName', defaultValue: 'TEST-site', description: 'Profile Name')
    
    string(name: 'component', defaultValue: 'TEST-component', description: 'Component Name')
    text(name: 'fileIncludePatterns', defaultValue: '**/*.txt')
    text(name: 'fileExcludePatterns', defaultValue: '''
      **/*tmp*
      **/.git
    ''')
    
    string(name: 'deployApp', defaultValue: 'TEST-app', description: 'Application Name')
    string(name: 'deployEnv', defaultValue: 'TEST-env', description: 'Environment Name')
    string(name: 'deployProc', defaultValue: 'TEST-app-process', description: 'Application Process Name')
        
    string(name: 'processName', defaultValue: 'TEST-globalProcess', description: 'Global Process Name')
    string(name: 'resourceName', defaultValue: 'TEST-resource', description: 'Resource Name')
    
    booleanParam(name: 'skip', defaultValue: false, description: 'Debug?')
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
        sh "echo '${env.BUILD_NUMBER}' > some_tmp_file.txt"
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
      // Publish artifacts and run Global Process
      step([$class: 'SerenaDAPublisher',
        skip: params.skip,
        siteName: params.siteName,

        baseDir: env.WORKSPACE,
        fileIncludePatterns: params.fileIncludePatterns,
        fileExcludePatterns: params.fileExcludePatterns,
        component: params.component,
        versionName: "${env.BUILD_NUMBER}-TEST",

        runProcess: true,
        processName: params.processName,
        resourceName: params.resourceName
      ])

      // artifacts archiver plugin usage
      step([$class: 'ArtifactArchiver', artifacts: '*.txt'])
    }
  }
  
}
