pipeline {
    agent {
        kubernetes {
            inheritFrom 'java'
        }
    }

    environment {
        REGISTRY = 'fatoura-docker.apps.dev.fatoura.gov.ma'
        PROJECT  = 'java-demo'
        TAG      = '1.0.0-SNAPSHOT'
    }

    stages {

        stage('Build') {
            steps {
                container('jnlp') {
                    sh "mvn clean package -DskipTests"
                }
            }
        }

        stage('Test') {
            steps {
                container('jnlp') {
                    sh "mvn test"
                }
            }
        }

        stage('Build Image') {
            when {
                branch 'master'
            }
            steps {
                container('jnlp') {
                    sh "buildah bud --tls-verify=false -t ${REGISTRY}/${PROJECT}:${TAG} -f Dockerfile ."
                }
            }
        }

        stage('Push Image') {
            when {
                branch 'master'
            }
            steps {
                container('jnlp') {
                    withCredentials([usernamePassword(credentialsId: 'nexus-creds-tmp', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
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
                container('jnlp') {
                    sh "buildah rmi ${REGISTRY}/${PROJECT}:${TAG} || true"
                }
            }
        }
    }
}
