node {
    checkout scm
    
    docker.withServer('tcp://172.17.94.121:2375') {
        def dev_erl_img = docker.build('dev-erl', '-f dockerfiles/Dockerfile.dev-erl .')
        dev_erl_img.inside() {
            sh 'echo Hello from'
        }
    }
}
