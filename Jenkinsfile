pipeline {
    agent {
        kubernetes {
            inheritFrom 'jnlp'
        }
    }

    environment {
        REGISTRY = 'nexus-nexus3.nexus.svc.cluster.local:8082'
        PROJECT  = 'demo'
        TAG      = ''
        PATH      = "/usr/lib/jvm/java-21-openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        JAVA_HOME = '/usr/lib/jvm/java-21-openjdk'
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
                container('jnlp') {
                    sh "buildah rmi ${REGISTRY}/${PROJECT}:${TAG} || true"
                }
            }
        }
    }
}
