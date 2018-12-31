pipeline {
    agent { 
        dockerfile {
            filename 'dockerfiles/Dockerfile.dev-erl'
            additionalBuildArgs '-t dev-erl'
            args '-p 7000:7000'
        }
    }
    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
    }
}
