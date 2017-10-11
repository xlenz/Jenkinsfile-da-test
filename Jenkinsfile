pipeline {
  agent { label 'master' }

  options { timeout(time: 5, unit: 'MINUTES') }

  parameters {
    string(name: 'siteName', defaultValue: 'TEST-site', description: 'Profile Name')
    
    string(name: 'componentName', defaultValue: 'TEST-component', description: 'Component Name')
    text(name: 'baseDir', defaultValue: '.', description: 'The base directory where the artifacts are located.')
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
    
    string(name: 'deployIf', defaultValue: 'true', description: 'Run Application process after publishing?')
    string(name: 'processIf', defaultValue: 'true', description: 'Run Global process after publishing?')
    string(name: 'runGlobalProcessIf', defaultValue: 'true', description: 'Run Global Process?')
    string(name: 'runApplicationProcessIf', defaultValue: 'true', description: 'Run Application Process?')
    
    booleanParam(name: 'deployUpdateJobStatus', defaultValue: true, description: 'Wait for a process completion?')
    booleanParam(name: 'processUpdateJobStatus', defaultValue: true, description: 'Wait for a process completion?')
    booleanParam(name: 'globalProcessUpdateJobStatus', defaultValue: true, description: 'Wait for a process completion?')
    booleanParam(name: 'appProcessUpdateJobStatus', defaultValue: false, description: 'Wait for a process completion?')
    
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
          deployIf: params.deployIf,
          deployUpdateJobStatus: params.deployUpdateJobStatus,
          deployApp: params.applicationName,
          deployEnv: params.environmentName,
          deployProc: params.applicationProcessName,
          deployProps: 'app-process-prop=some value'
        ])
        
        // Run Global Process
        script {
          def browsers = ['chrome', 'firefox', 'ie', 'edge']
          for (int i = 0; i < browsers.size(); ++i) {
            echo "Testing the ${browsers[i]} browser, ${i}"
            
            step([$class: 'RunGlobalProcessNotifier',
              siteName: params.siteName,

              runGlobalProcessIf: params.runGlobalProcessIf,
              updateJobStatus: params.globalProcessUpdateJobStatus,

              globalProcessName: params.globalProcessName,
              resourceName: params.resourceName,

              globalProcessProperties: """
                browser = ${browsers[i]}
                idx = ${i}
              """
            ])
          }
        }
        
        // Run Application Process
        step([$class: 'RunApplicationProcessNotifier',
          siteName: params.siteName,

          componentName: params.componentName,
          versionName: env.BUILD_NUMBER,
              
          runApplicationProcessIf: params.runApplicationProcessIf,
          updateJobStatus: params.appProcessUpdateJobStatus,

          applicationName: params.applicationName,
          environmentName: params.environmentName,
          applicationProcessName: params.applicationProcessName,
          applicationProcessProperties: 'prop1=value1'
        ])
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
        component: params.componentName,
        versionName: "${env.BUILD_NUMBER}-TEST",

        runProcess: true,
        processIf: params.processIf,
        processUpdateJobStatus: params.processUpdateJobStatus,
        processName: params.globalProcessName,
        resourceName: params.resourceName,
        processProps: 'prop=some'
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
