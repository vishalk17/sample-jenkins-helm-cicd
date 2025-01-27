// Send Email notification on below addresses

def emailRecipients = 'vishalkapadi95@gmail.com'
// for multiple recipients:  def emailRecipients = 'xyz@gmail.com,abc@gmail.com'

pipeline {
    agent {
        label 'master'
    }
    environment {
    //    CHATS_TOKEN = credentials('hangout-token') // Use google chat jenkins token
        JENKINS_URL = "${env.JENKINS_URL}"    // Jenkins URL
        BUILD_URL = "${env.BUILD_URL}"    // Build URL
        CONSOLE_URL = "${BUILD_URL}/console"    // Console output URL
        BLUE_OCEAN_URL = "${JENKINS_URL}/blue/organizations/jenkins/${JOB_NAME}/detail/${JOB_NAME}/${BUILD_NUMBER}/pipeline" // Blue Ocean URL
        JOB_NAME = "${env.JOB_NAME}"    // Job Name
        BUILD_NUMBER = "${env.BUILD_NUMBER}"    // Build Number
        NEXUS_URL = 'http://localhost:8092/repository/helm-chart-repo/'    // helm chart repo or artifact, here I have hosted on sonatype nexus
        NEXUS_CREDENTIAL_ID = 'nexus-credential' // Nexus credential for helm chart artifactory or repo
        // add helm charts src directories below
        HELM_SRC_DIRECTORIES = '''
            bitnami/apache
            bitnami/grafana-loki
            bitnami/fluentd
            bitnami/mysql
        '''
    }

    stages {
        stage('Prepare Environment') {
            steps {
                echo 'Preparing the environment'
                sh "rm -rf */*.tgz"
            }
        }

        stage('Package Helm Charts') {
            steps {
                script {
                    for (srcDir in env.HELM_SRC_DIRECTORIES.split()) {
                        sh "cd $srcDir && microk8s helm3 package ."
                        sh "tree"
                    }
                }
            }
        }

        stage('Upload to Nexus Helm Repo') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: env.NEXUS_CREDENTIAL_ID, usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                        for (srcDir in env.HELM_SRC_DIRECTORIES.split()) {
                            helmFiles = sh(script: "ls $srcDir/*.tgz", returnStdout: true).trim().split('\n')

                            for (helmFile in helmFiles) {
                                helmPackageName = helmFile.tokenize('/').last()
                                uploadCommand = "curl -u $NEXUS_USERNAME:$NEXUS_PASSWORD $NEXUS_URL --upload-file $helmFile"
                                sh uploadCommand
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Helm packages successfully compiled and uploaded to Nexus Helm chart repo.'
            script {
                currentBuild.result = 'SUCCESS'
            //    bitbucketStatusNotify(buildState: 'SUCCESSFUL')
            //    hangoutsNotifySuccess token: "$CHATS_TOKEN", threadByJob: false
                    emailext subject: "Helm packages successfully compiled and uploaded: ${JOB_NAME} #${BUILD_NUMBER}",
                        mimeType: 'text/html',
                        body: """
                        The job has succeeded.<br>
                        <a href='${BUILD_URL}'>Click here</a> to view the job in Jenkins.<br>
                        <a href='${CONSOLE_URL}'>Click here</a> to view the console output.<br>
                        <a href='${BLUE_OCEAN_URL}'>Click here</a> to view the Blue Ocean URL.
                        """,
                        to: emailRecipients
            }
        }
        failure {
            echo 'Failed to compile and upload Helm packages.'
            script {
                currentBuild.result = 'FAILURE'
             //   bitbucketStatusNotify(buildState: 'FAILED')
             //   hangoutsNotifySuccess token: "$CHATS_TOKEN", threadByJob: false
                    emailext subject: "Failed to compile and upload Helm packages: ${JOB_NAME} #${BUILD_NUMBER}",
                        mimeType: 'text/html',
                        body: """
                        The job has succeeded.<br>
                        <a href='${BUILD_URL}'>Click here</a> to view the job in Jenkins.<br>
                        <a href='${CONSOLE_URL}'>Click here</a> to view the console output.<br>
                        <a href='${BLUE_OCEAN_URL}'>Click here</a> to view the Blue Ocean URL.
                        """,
                        to: emailRecipients
            }
        }
    }
}
