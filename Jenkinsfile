def labelArm = "docker-ansible${UUID.randomUUID().toString()}"
def labelx86_64 = "docker-ansible${UUID.randomUUID().toString()}"


stage('Build') {
    podTemplate(
            label: labelArm,
            containers: [
                    containerTemplate(name: 'ansible',
                            image: 'voight/docker-ansible:latest',
                            alwaysPullImage: false,
                            ttyEnabled: true,
                            command: 'cat',
                            envVars: [containerEnvVar(key: 'DOCKER_HOST', value: "unix:///var/run/docker.sock")],
                            privileged: false),
                    containerTemplate(name: 'jnlp', image: 'jenkins/inbound-agent:latest-jdk11', args: '${computer.jnlpmac} ${computer.name}'),
            ],
            volumes: [
                    hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
                    hostPathVolume(hostPath: "$(pwd)", mountPath: '/ansible/playbooks')
            ],
            nodeSelector: 'kubernetes.io/arch=arm64'
    ) {
        node(labelArm) {
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
                    withCredentials([string(credentialsId: 'ansible-vault-pwd', variable: 'ansiblevaultpwd')]) {
                        sh "sh -c 'echo ${ansiblevaultpwd} > vaultpwd'"
                        sh "ansible-playbook --vault-pwd-file=./vaultpwd --version"
                        sh "rm vaultpwd"
                    }
                }
            }
        }
    }

    podTemplate(
            label: labelx86_64,
            containers: [
                    containerTemplate(name: 'ansible',
                            image: 'voight/docker-ansible:latest',
                            alwaysPullImage: false,
                            ttyEnabled: true,
                            command: 'cat',
                            envVars: [containerEnvVar(key: 'DOCKER_HOST', value: "unix:///var/run/docker.sock")],
                            privileged: true),
                    containerTemplate(name: 'jnlp', image: 'jenkins/inbound-agent:latest-jdk11', args: '${computer.jnlpmac} ${computer.name}'),
            ],
            volumes: [
                    hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
                    hostPathVolume(hostPath: "$(pwd)", mountPath: '/ansible/playbooks')
            ],
            nodeSelector: 'kubernetes.io/arch=amd64'
    ) {
        node(labelx86_64) {
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
                    withCredentials([string(credentialsId: 'ansible-vault-pwd', variable: 'ansiblevaultpwd')]) {
                        sh "sh -c 'echo ${ansiblevaultpwd} > vaultpwd'"
                        sh "ansible-playbook --vault-pwd-file=./vaultpwd -e namespace=blah ./playbook.yaml"
                        sh "rm vaultpwd"
                    }
                }
            }
        }
    }
}
