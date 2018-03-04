env.ghprbGhRepository = env.ghprbGhRepository ?: 'http://git.engineering.redhat.com/git/users/nkondras/skt.git'
env.ghprbActualCommit = env.ghprbActualCommit ?: 'master'
env.ghprbPullAuthorLogin = env.ghprbPullAuthorLogin ?: ''

// Needed for podTemplate()
env.SKT_TAG = env.SKT_TAG ?: 'stable'
env.SLAVE_TAG = env.SLAVE_TAG ?: 'stable'

env.DOCKER_REPO_URL = env.DOCKER_REPO_URL ?: 'docker-registry.default.svc:5000'
env.OPENSHIFT_NAMESPACE = env.OPENSHIFT_NAMESPACE ?: 'continuous-infra'
env.OPENSHIFT_SERVICE_ACCOUNT = env.OPENSHIFT_SERVICE_ACCOUNT ?: 'jenkins'

// Execution ID for this run of the pipeline
executionID = UUID.randomUUID().toString()

// Pod name to use
podName = 'ark-' + executionID


// Existing skt vars from Dzickus Jenkinsfile
def sktrc
def lockfile = "/home/worker/.sktrh7.lock"
def cfgfile = "kconf"
def rcfile = "sktrc"

properties([
        parameters(
                [
                        string(defaultValue: 'stable', description: 'Tag for slave image', name: 'SLAVE_TAG'),
                        string(defaultValue: 'stable', description: 'Tag for rpmbuild-rhel image', name: 'SKT_TAG'),
                        string(defaultValue: 'docker-registry.default.svc:5000', description: 'Docker repo url for Openshift instance', name: 'DOCKER_REPO_URL'),
                        string(defaultValue: 'continuous-infra', description: 'Project namespace for Openshift operations', name: 'OPENSHIFT_NAMESPACE'),
                        string(defaultValue: 'jenkins', description: 'Service Account for Openshift operations', name: 'OPENSHIFT_SERVICE_ACCOUNT'),
                ]
        )
])


podTemplate(name: podName,
        label: podName,
        cloud: 'openshift',
        serviceAccount: OPENSHIFT_SERVICE_ACCOUNT,
        idleMinutes: 0,
        namespace: OPENSHIFT_NAMESPACE,

        containers: [
                // This adds the custom slave container to the pod. Must be first with name 'jnlp'
                containerTemplate(name: 'jnlp',
                        image: DOCKER_REPO_URL + '/' + OPENSHIFT_NAMESPACE + '/jenkins-provisioning-slave:' + SLAVE_TAG,
                        ttyEnabled: false,
                        args: '${computer.jnlpmac} ${computer.name}',
                        envVars: [
                                envVar(key: 'GIT_SSL_NO_VERIFY', value: 'true')
                        ],
                        command: '',
                        workingDir: '/workDir'),
                // This adds the rpmbuild test container to the pod.
                containerTemplate(name: 'skt',
                        alwaysPullImage: true,
                        image: DOCKER_REPO_URL + '/' + OPENSHIFT_NAMESPACE + '/skt:' + SKT_TAG,
                        ttyEnabled: true,
                        command: 'cat',
                        privileged: true,
                        workingDir: '/workDir'),
        ],
        volumes: [emptyDirVolume(memory: false, mountPath: '/sys/class/net')])
        {
            node(podName) {

                def currentStage = ""

                ansiColor('xterm') {
                    timestamps {
                        // We need to set env.HOME because the openshift slave image
                        // forces this to /home/jenkins and then ~ expands to that
                        // even though id == "root"
                        // See https://github.com/openshift/jenkins/blob/master/slave-base/Dockerfile#L5
                        //
                        // Even the kubernetes plugin will create a pod with containers
                        // whose $HOME env var will be its workingDir
                        // See https://github.com/jenkinsci/kubernetes-plugin/blob/master/src/main/java/org/csanchez/jenkins/plugins/kubernetes/KubernetesLauncher.java#L311
                        //
                        env.HOME = "/root"
                        //
                        try {
                            // Set our current stage value
                            currentStage = "run-skt"
                            stage(currentStage) {

                                // SCM
                                dir('skt') {
                                    // Checkout out skt
                                    checkout([$class: 'GitSCM', branches: [[name: env.ghprbActualCommit]],
                                              doGenerateSubmoduleConfigurations: false,
                                              extensions                       : [],
                                              submoduleCfg                     : [],
                                              userRemoteConfigs                : [
                                                      [refspec:
                                                               '+refs/heads/*:refs/remotes/origin/*  +refs/pull/*:refs/remotes/origin/pr/* ',
                                                       url: "${env.ghprbGhRepository}"]
                                              ]
                                    ])
                                }
                                // Execute run skt
                                executeInContainer(currentStage, "skt", "python ./skt/skt.py --help")
                            }
                        } catch (e) {
                            // Set build result
                            currentBuild.result = 'FAILURE'

                            // Report the exception
                            echo "Error: Exception from " + currentStage + ":"
                            echo e.getMessage()

                            // Throw the error
                            throw e

                        } finally {
                            currentBuild.displayName = "Build#: ${env.BUILD_NUMBER}"
                            currentBuild.description = "Result: ${currentBuild.currentResult}"
                        }
                    }
                }
            }
        }

def executeInContainer(String stageName, String containerName, String script) {
    //
    // Kubernetes plugin does not let containers inherit
    // env vars from host. We force them in.
    //
    containerEnv = env.getEnvironment().collect { key, value -> return key+'='+value }
    sh "mkdir -p ${stageName}"
    try {
        withEnv(containerEnv) {
            container(containerName) {
                sh script
            }
        }
    } catch (err) {
        throw err
    } finally {
        sh "mv -vf logs ${stageName}/logs || true"
    }
}