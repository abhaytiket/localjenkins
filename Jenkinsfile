pipeline {
    agent { 
        docker { 
            image 'gcr.io/bazel-public/bazel:6.4.0' 
            args '--platform=linux/amd64'
        } 
    }
    stages {
        stage('build') {
            steps {
                sh 'bazel --version'
                sh 'bazel build //src/app1/hello'
            }
        }
    }
}
