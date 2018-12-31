node {
    checkout scm
    
    docker.withServer('tcp://172.17.94.121:2375') {
        def dev_erl_img = docker.build('erlang-alpine-dev', '-f dockerfiles/Dockerfile.erlang-alpine-dev .')
        dev_erl_img.withRun('-p 7000:7000') {
            sh 'echo Hello from erlange-alpine-dev'
        }
    }
}
