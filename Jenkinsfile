def labelArm = "mvn-jdk-build-arm64${UUID.randomUUID().toString()}"
def labelx86_64 = "mvn-jdk-build-x86_64${UUID.randomUUID().toString()}"


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
                    hostPathVolume(hostPath: '/etc/ssl/certs', mountPath: '/certs/ca'),
                    hostPathVolume(hostPath: '/etc/ssl/certs', mountPath: '/certs/server'),
                    hostPathVolume(hostPath: '/etc/ssl/certs', mountPath: '/certs/client')
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
                    withCredentials([string(credentialsId: 'ansible-vault-pwd', variable: 'ansible-vault-pwd')]) {
                        sh "--version"
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
                    hostPathVolume(hostPath: '/etc/ssl/certs', mountPath: '/certs/ca'),
                    hostPathVolume(hostPath: '/etc/ssl/certs', mountPath: '/certs/client'),
                    hostPathVolume(hostPath: '/etc/ssl/certs', mountPath: '/certs/server')
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
                    withCredentials([string(credentialsId: 'ansible-vault-pwd', variable: 'ansible-vault-pwd')]) {
                        sh "--version"
                    }
                }
            }
        }
    }
}
