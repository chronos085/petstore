pipeline {
    agent none
    stages {
        /*stage('Build Proxy from Spec') {
            agent {
                docker { image 'chronos085/node-apigee:8-alpine' }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]){
                    sh 'openapi2apigee generateApi proxy -s openapi.yaml -d apigee'
                    sh 'rm -rf apigee/proxy/apiproxy.zip'
                    sh 'apigeelint -s apigee/proxy/apiproxy -f table.js'
                    sh 'git add -A apigee'
                    sh 'git commit -m "proxy commit"'
                    sh 'git tag -a petstore-$BUILD_NUMBER -m "Jenkins"'
                    sh 'git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/chronos085/petstore.git HEAD:master  --tags'
                }
            }
        }*/
        stage('Build Proxy to Maven') {
            agent {
                docker { image 'chronos085/maven-apigee:3-alpine' }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]){
                    sh 'mvn --version'
                    sh 'git clone https://$GIT_USERNAME:$GIT_PASSWORD@github.com/chronos085/ci-apigee.git'
                    sh 'ls'
                    sh 'cp -r ci-apigee/maven/ apigee/proxy/'
                    sh 'ls'
                }
            }
        }
    }
}
