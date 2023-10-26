pipeline {
    agent { docker { image 'gcr.io/bazel-public/bazel:latest' } }
    stages {
        stage('build') {
            steps {
                sh 'bazel --version'
            }
        }
    }
}
