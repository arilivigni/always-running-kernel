env.ghprbGhRepository = env.ghprbGhRepository ?: 'arilivigni/always-running-kernel'
env.ghprbActualCommit = env.ghprbActualCommit ?: 'master'
env.ghprbPullAuthorLogin = env.ghprbPullAuthorLogin ?: ''

// Needed for podTemplate()
env.SKT_TAG = env.SKT_TAG ?: 'stable'

env.DOCKER_REPO_URL = env.DOCKER_REPO_URL ?: '172.30.254.79:5000'
env.OPENSHIFT_NAMESPACE = env.OPENSHIFT_NAMESPACE ?: 'continuous-infra'
env.OPENSHIFT_SERVICE_ACCOUNT = env.OPENSHIFT_SERVICE_ACCOUNT ?: 'jenkins'
def sktrc
def lockfile = "/home/worker/.sktrh7.lock"
def cfgfile = "kconf"
def rcfile = "sktrc"

properties([
        parameters([
                string(defaultValue: 'git://git.app.eng.bos.redhat.com/rhel7.git', description: 'base repo url', name: 'baserepo'),
                string(defaultValue: 'master', description: 'base repo ref', name: 'ref'),
                string(defaultValue: 'http://git.engineering.redhat.com/git/users/nkondras/fmk-files.git/plain/kernel-3.10.0-x86_64-debug.config', description: 'URL to fetch kernel config from', name: 'baseconfig'),
                string(defaultValue: '', description: 'Space separated list of patchwork urls', name: 'patchwork'),
                string(defaultValue: '', description: 'Additional make arguments', name: 'makeopts'),
                string(defaultValue: '', description: 'Comma separated list of emails to send reports to', name: 'emails')])
])


podTemplate(name: podName,
        label: podName,
        cloud: 'openshift',
        serviceAccount: OPENSHIFT_SERVICE_ACCOUNT,
        idleMinutes: 0,
        namespace: OPENSHIFT_NAMESPACE,

        containers: [
                // This adds the rpmbuild test container to the pod.
                containerTemplate(name: 'skt',
                        alwaysPullImage: true,
                        image: DOCKER_REPO_URL + '/' + OPENSHIFT_NAMESPACE + '/skt:' + SKTBUILD_TAG,
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
                        dir('ci-pipeline') {
                            // Checkout our ci-pipeline repo based on the value of env.ghprbActualCommit
                            checkout([$class: 'GitSCM', branches: [[name: env.ghprbActualCommit]],
                                      doGenerateSubmoduleConfigurations: false,
                                      extensions                       : [],
                                      submoduleCfg                     : [],
                                      userRemoteConfigs                : [
                                              [refspec:
                                                       '+refs/heads/*:refs/remotes/origin/*  +refs/pull/*:refs/remotes/origin/pr/* ',
                                               url: "https://github.com/${env.ghprbGhRepository}"]
                                      ]
                            ])
                        }
                        sh '''
                            ./skt --help
                        '''
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
                    currentBuild.description = currentBuild.result
                }
            }
        }
    }
}