pipeline {
    agent any
    stages {
        stage('initialize') {
            steps {
                echo 'Remove the previous one!'
            }
            post {
                always {
                    deleteDir() /* clean up our workspace */
                }
            }
        }
        stage('build') {
            steps {
                echo 'Hello build stage!'
                echo "Hello build ${env.BUILD_ID}"
                //checkout([$class: 'GitSCM', branches: [[name: '*/master']],
                //    userRemoteConfigs: [[url: 'https://github.com/Merlion-Crew/python-docs-hello-world.git/']]])

                sh """
                    ls
                    pip3 install -r requirements.txt
                """
            }
        }
        stage('test') {
            steps {
                echo 'Hello test stage!'
                sh """
                    python3 -m pytest
                    python3 -m coverage xml test_flask.py
                """
            }
            post {
                always {
                    echo 'This will always run'
                    step([$class: 'CoberturaPublisher',
                                   autoUpdateHealth: false,
                                   autoUpdateStability: false,
                                   coberturaReportFile: 'coverage.xml',
                                   failNoReports: false,
                                   failUnhealthy: false,
                                   failUnstable: false,
                                   maxNumberOfBuilds: 10,
                                   onlyStable: false,
                                   sourceEncoding: 'ASCII',
                                   zoomCoverageChart: false])
                }
                success {
                    echo 'This will run only if successful'
                }
                failure {
                    echo 'This will run only if failed'
                }
            }
        }
        stage('packaging') {
            steps {
                echo 'Packaging!'
                sh "zip -r deployment-${BUILD_NUMBER}.zip ."
            }
        }
        // stage('Approval') {
        //     // no agent, so executors are not used up when waiting for approvals
        //     agent none
        //     steps {
        //         script {
        //             def deploymentDelay = input id: 'Deploy', message: 'Deploy to production?', submitter: 'toanhuyn', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
        //             sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
        //         }
        //     }
        // }
        stage('prod') {
            steps {
                echo 'Deploying!'
                sh '''
                    curl -X POST -u '\$upskilltoan-app-service':RhxlfkpJYYXBufdoxT3ZfCYSB2ZPQr59MJvBpQtwKAiRE7nmEucAlWxcJRa6 --data-binary @"deployment-${BUILD_NUMBER}.zip" https://upskilltoan-app-service.scm.azurewebsites.net/api/zipdeploy
                '''
            }
            post {
                always {
                    emailext body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
                        recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                        subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
                }
            }
        }
    }
}