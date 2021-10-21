GString label = "docker-ansible${UUID.randomUUID().toString()}"

stage('Build') {

    podTemplate(
            label: label,
            containers: [
                    containerTemplate(name: 'ansible',
                            image: 'voight/ansible-k8s:1.0',
                            alwaysPullImage: false,
                            ttyEnabled: true,
                            command: 'cat',
                            envVars: [containerEnvVar(key: 'DOCKER_HOST', value: "unix:///var/run/docker.sock")],
                            privileged: false),
                    containerTemplate(name: 'jnlp', image: 'jenkins/inbound-agent:latest-jdk11', args: '${computer.jnlpmac} ${computer.name}'),
            ],
            volumes: [
                    hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
            ]
    ) {
        node(label) {
            stage('Git Checkout') {
                def scmVars = checkout([
                        $class           : 'GitSCM',
                        userRemoteConfigs: scm.userRemoteConfigs,
                        branches         : scm.branches,
                        extensions       : scm.extensions
                ])

                // used to create the Docker image
                env.GIT_BRANCH = scmVars.GIT_BRANCH
                env.GIT_COMMIT = scmVars.GIT_COMMIT
            }
            stage('Push') {
                container('ansible') {
                    sh 'env'
                    sh 'ls -l'
                    withCredentials([string(credentialsId: 'ansible-vault-pwd', variable: 'ansiblevaultpwd')]) {
                        sh "sh -c 'echo ${ansiblevaultpwd} > vaultpwd'"
                        dir("ansible") {
                            sh "ansible-playbook --vault-password-file=./vaultpwd ./playbook.yaml"
                            sh "rm vaultpwd"
                        }
                    }
                }
            }
        }
    }

}
