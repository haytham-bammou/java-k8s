pipeline {
    agent { label 'java' }

    environment {
        REGISTRY = 'nexus-nexus3.nexus.svc.cluster.local:8082'
        PROJECT  = 'demo'
        TAG      = ''
    }

    stages {
        stage('Init') {
            steps {
                container('java') {
                    script {
                        TAG = sh(script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout", returnStdout: true).trim()
                    }
                }
            }
        }

        stage('Build') {
            steps {
                container('java') {
                    sh "mvn clean package -DskipTests"
                }
            }
        }

        stage('Test') {
            steps {
                container('java') {
                    sh "mvn test"
                }
            }
        }

        stage('Build Image') {
            when {
                branch 'master'
            }
            steps {
                container('java') {
                    sh "buildah bud --tls-verify=false -t ${REGISTRY}/${PROJECT}:${TAG} -f Dockerfile ."
                }
            }
        }

        stage('Push Image') {
            when {
                branch 'master'
            }
            steps {
                container('java') {
                    withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh "buildah login --tls-verify=false -u \$USER -p \$PASS ${REGISTRY}"
                        sh "buildah push --tls-verify=false ${REGISTRY}/${PROJECT}:${TAG}"
                    }
                }
            }
        }

        stage('Cleanup') {
            when {
                branch 'master'
            }
            steps {
                container('java') {
                    sh "buildah rmi ${REGISTRY}/${PROJECT}:${TAG} || true"
                }
            }
        }
    }
}
