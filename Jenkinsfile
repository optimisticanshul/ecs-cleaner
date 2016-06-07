final def String ECR_REGISTRY = '542640492856.dkr.ecr.us-west-2.amazonaws.com'
final def String ECR_REPO     = 'ecs-cleaner'
final def String ECR_REGION   = 'us-west-2'  // For the above repo, not for the clean target

def doCheckout() {
    stage 'checkout'
    checkout scm

    sh('git rev-parse --short HEAD > GIT_COMMIT')
    return readFile('GIT_COMMIT').trim()
}

def doErrata() {
    stage 'errata'
    sh "sudo puppet agent --enable; sudo puppet agent -tdv --tags=bash; sudo puppet agent --disable"
}

def doBuild(tag) {
    stage 'build'
    sh "docker build -t ${tag} ."
}

def doPush(tag) {
    stage 'push'
    sh './ci/ecr-login'
    sh "docker push $tag"
}

def doPromote(tag) {
    sh "docker tag ${tag} master"
    sh "docker push master"
}

node('trusty && vendci') {
    wrap([$class: 'AnsiColorBuildWrapper']) {
        wrap([$class: 'TimestamperBuildWrapper']) {
            sshagent(['80e76a2a-650e-4027-a4cd-f19bb4c9a439']) {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    credentialsId: 'ecr-access',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    def commit = doCheckout()
                    def tag = "${ECR_REGISTRY}/${ECR_REPO}:${commit}"

                    withEnv([
                            "GIT_COMMIT=${commit}",
                            "AWS_DEFAULT_REGION=${ECR_REGION}"
                    ]) {
                        echo "Building for ${branch}/${commit}: ${tag}"

                        doErrata()
                        doBuild(tag)
                        doPush(tag)

                        if (env.BRANCH_NAME === 'master') {
                            doPromote(tag)
                        }
                    }
                }
            }
        }
    }
}
