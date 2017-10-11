pipeline {
  agent { label 'master' }

  options { timeout(time: 1, unit: 'MINUTES') }

  parameters {
    string(name: 'siteName', defaultValue: 'TEST-site', description: 'Profile Name')
    
    string(name: 'componentName', defaultValue: 'TEST-component', description: 'Component Name')
    text(name: 'fileIncludePatterns', defaultValue: '**/*.txt')
    text(name: 'fileExcludePatterns', defaultValue: '''
      **/*tmp*
      **/.git
    ''')
    
    string(name: 'statusName', defaultValue: 'TEST-versionStatus', description: 'Version Status Name')
    
    string(name: 'applicationName', defaultValue: 'TEST-app', description: 'Application Name')
    string(name: 'environmentName', defaultValue: 'TEST-env', description: 'Environment Name')
    string(name: 'applicationProcessName', defaultValue: 'TEST-app-process', description: 'Application Process Name')
        
    string(name: 'globalProcessName', defaultValue: 'TEST-globalProcess', description: 'Global Process Name')
    string(name: 'resourceName', defaultValue: 'TEST-resource', description: 'Resource Name')
    
    booleanParam(name: 'skip', defaultValue: false, description: 'Skip publish artifacts step?')
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
        // Publish artifacts, add version status and run Application Process
        step([$class: 'SerenaDAPublisher',
          skip: params.skip,
          siteName: params.siteName,

          baseDir: env.WORKSPACE,
          fileIncludePatterns: params.fileIncludePatterns,
          fileExcludePatterns: params.fileExcludePatterns,
          component: params.componentName,
          versionName: env.BUILD_NUMBER,
          
          addStatus: true,
          statusName: params.statusName,

          deploy: true,
          deployApp: params.applicationName,
          deployEnv: params.environmentName,
          deployProc: params.applicationProcessName
        ])
        
        // Run Global Process
        step([$class: 'RunGlobalProcessNotifier',
          siteName: params.siteName,

          globalProcessName: params.globalProcessName,
          resourceName: params.resourceName
        ])
        
        // Run Application Process
        step([$class: 'RunApplicationProcessNotifier',
          siteName: params.siteName,

          componentName: params.componentName,
          versionName: env.BUILD_NUMBER,

          applicationName: params.applicationName,
          environmentName: params.environmentName,
          applicationProcessName: params.applicationProcessName
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

  // using post-build actions
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
        processName: params.globalProcessName,
        resourceName: params.resourceName
      ])
      
      // Set component version status
      step([$class: 'UpdateComponentVersionStatusNotifier',
        siteName: params.siteName,

        action: 'ADD',
        componentName: params.componentName,
        versionName: "${env.BUILD_NUMBER}-TEST",
        statusName: params.statusName
      ])

      // Remove component version status
      step([$class: 'UpdateComponentVersionStatusNotifier',
        siteName: params.siteName,

        action: 'REMOVE',
        componentName: params.componentName,
        versionName: env.BUILD_NUMBER,
        statusName: params.statusName
      ])

      // artifacts archiver plugin usage
      step([$class: 'ArtifactArchiver', artifacts: '*.txt'])
    }
  }
  
}
