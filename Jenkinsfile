pipeline {
      agent any

      tools {nodejs "NodeJS"}


      parameters{
          string(name: 'SPEC', defaultValue:"cypress/integration/google-search.js", description: "Enter the cypress script path that you want to execute")
          choice(name: 'BROWSER', choices:['electron', 'chrome', 'edge', 'firefox'], description: "Select the browser to be used in your cypress tests")
          booleanParam(name: 'skip_test', defaultValue: false, description: 'Set to true to skip the test stage')
          booleanParam(name: 'skip_sonar', defaultValue: false, description: 'Set to true to skip the SonarQube stage')
          booleanParam(name: 'skip_jmeter', defaultValue: false, description: 'Set to true to skip the SonarQube stage')
      }

      options {
              ansiColor('xterm')
      }

     
      stages {
        stage('Run automated tests'){
            when { expression { params.skip_test != true } }
            steps {
              echo "Running automated tests"
                sh 'npm prune'
                sh 'npm cache clean --force'
                sh 'npm i'
                sh 'npm install --save-dev mochawesome mochawesome-merge mochawesome-report-generator'
                sh 'npm run jenkins'
            }
            post {
                success {
                    publishHTML (
                        target : [
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'mochawesome-report',
                            reportFiles: 'mochawesome.html',
                            reportName: 'My Reports',
                            reportTitles: 'The Report'])

                }
            }
        }
        
        stage('SonarQube analysis') {
          when { expression { params.skip_sonar != true } }
          steps {
            script {
                      scannerHome = tool 'sonar-scanner';
                 }
            withSonarQubeEnv('sonar-scanner') { // If you have configured more than one global server connection, you can specify its name
            sh "${scannerHome}/bin/sonar-scanner"
            }
          }
        }

        stage('JMeter Test') {
            when { expression { params.skip_jmeter != true } } 
            steps {
                script {
                    // Path to the JMeter installation directory
                    def jmeterHome = '/usr/share/jmeter'

                    // Path to the JMeter test script
                    def jmeterScript = './testPlanHome.jmx'

                    // Execute JMeter test
                    sh "${jmeterHome}/bin/jmeter -n -t ${jmeterScript} -l result.jtl"
                }
            }
            post {
                always {
                    // Archive JTL result file
                    archiveArtifacts 'result.jtl'
                }
                success {
                    // Publish JMeter report using Performance plugin
                    perfReport filterRegex:'', sourceDataFiles: 'result.jtl'
                }
            }
        }

        stage('Perform manual testing...'){
            steps {
                timeout(activity: true, time: 5) {
                    input 'Proceed to production?'
                }
           }
        }

        stage('Finished (Release to production)') {
            steps {
            // similar procedure as in the 'Build/ Deploy to staging' stage, suppressed here for cost saving purposes
                echo "Done!"
           }
        }
    }
}