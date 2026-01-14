pipeline {
    agent any

    stages {
        // 2.1 La phase Test
        stage('Test') {
            steps {
                // Change: bat 'gradlew' -> sh './gradlew'
                sh 'chmod +x gradlew' // Ensures the wrapper has execution permissions
                sh './gradlew test'
            }
            post {
                always {
                    junit 'build/test-results/test/*.xml'
                    cucumber 'reports/*.json'
                }
            }
        }

        // 2.2 La phase Code Analysis
        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh './gradlew sonar'
                }
            }
        }


        // 2.3 La phase Code Quality
        stage('Code Quality') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // 2.4 La phase Build
        stage('Build') {
            steps {
                sh './gradlew assemble'
                sh './gradlew javadoc'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'build/libs/*.jar, build/docs/javadoc/**'
                }
            }
        }

        // 2.5 La phase Deploy
        stage('Deploy') {
            steps {
                sh './gradlew publish'
            }
        }
    }

    post {
        success {
            mail to: 'kh_benferhat@esi.dz',
                 subject: "Success: ${currentBuild.fullDisplayName}",
                 body: "The build and deploy were successful."

            slackSend color: 'good',
                      channel: 'tp_gradle',
                      message: "Build Success: ${currentBuild.fullDisplayName} (<${env.BUILD_URL}|Open>)"
        }
        failure {
            mail to: 'kh_benferhat@esi.dz',
                 subject: "Failed: ${currentBuild.fullDisplayName}",
                 body: "The pipeline failed in stage: ${env.STAGE_NAME}"

            slackSend color: 'danger',
                      channel: 'tp_ogl_gradle',
                      message: "Build Failed: ${currentBuild.fullDisplayName} in stage ${env.STAGE_NAME} (<${env.BUILD_URL}|Open>)"
        }
    }
}