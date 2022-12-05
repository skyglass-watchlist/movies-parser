def imageName = 'skyglass/movies-parser'
def registry = 'https://registry.hub.docker.com'

node('workers'){
    stage('Checkout'){
        checkout scm
    }

    def imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")

    stage('Pre-integration Tests'){
        parallel(
            'Quality Tests': {
                imageTest.inside{
                    sh 'golint'
                }
            },
            'Unit Tests': {
                imageTest.inside('-u root:root'){
                    sh 'go test'
                }
            }
        )
    }

    stage('Build'){
        docker.build(imageName)
    }

    stage('Push'){
        docker.withRegistry(registry, 'dockerHubCredentials') {
            docker.image(imageName).push(commitID())

            if (env.BRANCH_NAME == 'develop') {
                docker.image(imageName).push('develop')
            }
        }
    }

    stage('Deploy'){
        if(env.BRANCH_NAME == 'develop' || env.BRANCH_NAME == 'preprod'){
            build job: "watchlist-deployment/${env.BRANCH_NAME}"
        }

        if(env.BRANCH_NAME == 'master'){
            timeout(time: 2, unit: "HOURS") {
                input message: "Approve Deploy?", ok: "Yes"
            }
            build job: "watchlist-deployment/master"
        }
    }
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim()
    sh 'rm .git/commitID'
    commitID
}