pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'python3 -m py_compile sources/add2vals.py sources/calc.py'
                // stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                apt-get update && apt-get install -y python3-pip
                python3 -m pip install --upgrade pip --break-system-packages
                python3 -m pip install pytest --break-system-packages
                '''
            }
        }

        stage('Test') {
            steps {
                sh 'pytest --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('CodeScanning'){

            environment {
                SONAR_HOME = tool name: 'sonar-scan' 
            }

            steps {
                withSonarQubeEnv('sonar-qube'){
                    sh '''$SONAR_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=APP \
                        -Dsonar.projectName=pyinstallerapp \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=. \
                        -Dsonar.sourceEncoding=UTF-8
                    '''
                }


            }
        }

        stage('CheckQualityGate'){

            steps {

                timeout(time: 60, unit: 'SECONDS') {
                    waitForQualityGate  abortPipeline: true
                }
            }
        }


        stage('Upload Python Artifact to Nexus Raw Repo') {
                steps {
                    script {
                        def version = '13'
                        def projectName = 'python-app'
                        def artifactFile = "${projectName}-${version}.tar.gz"
                        def groupPath = 'com/example/python-app'
                        def nexusUrl = 'http://nexus:8081'
                        def repository = 'python-app'
                        def credentialsId = 'nexus-creds'

                        // Compress project source
                        sh "tar -czf ${artifactFile} sources/*.py"

                        // Get credentials from Jenkins (username and password)
                        withCredentials([usernamePassword(credentialsId: credentialsId, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                            def uploadUrl = "${nexusUrl}/repository/${repository}/${groupPath}/${version}/${artifactFile}"
                            
                            // Upload using curl
                            sh """
                                curl -u $NEXUS_USER:$NEXUS_PASS --upload-file ${artifactFile} ${uploadUrl}
                            """
                        }
                    }
                }
        }
       




        
    }
}

 // stage('Upload Python Artifact as Fake Maven Artifact') {
        //     steps {
        //         script {
        //             def version = '13'
        //             def projectName = 'python-app'
        //             def artifactFile = "${projectName}-${version}.tar.gz"
        //             def repository = 'maven-releases'  // MUST be a Maven repo
        //             def credentialsId = 'nexus-creds'

        //             // Compress project source
        //             sh "tar -czf ${artifactFile} sources/*.py"

        //             // Upload to Maven repo with custom 'type' (still tricks Nexus into accepting it)
        //             nexusArtifactUploader(
        //                 nexusVersion: 'nexus3',
        //                 protocol: 'http',
        //                 nexusUrl: 'http://nexus:8081',
        //                 groupId: 'com.example',
        //                 version: version,
        //                 repository: repository,
        //                 credentialsId: credentialsId,
        //                 artifacts: [[
        //                     artifactId: projectName,
        //                     classifier: '',
        //                     file: artifactFile,
        //                     type: 'tar.gz'
        //                 ]]
        //             )
        //         }
        //     }
        // }