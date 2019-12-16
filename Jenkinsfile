pipeline {
    agent none
    stages {
        stage('Build Proxy to Spec') {
            agent {
                docker { image 'chronos085/node-apigee:8-alpine' }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: github, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]){
                    sh 'openapi2apigee generateApi proxy -s openapi.yaml -d apigee'
                    sh 'apigeelint -s apigee/proxy/apiproxy -f table.js'
                    sh 'git add -A apigee'
                    sh 'git commit -m "proxy commit"'
                    sh 'git tag -a petstore -m "Jenkins"'
                    sh 'git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/chronos085/petstore.git HEAD:master  --tags'
                }
            }
        }
    }
}
