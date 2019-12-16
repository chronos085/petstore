pipeline {
    agent none
    stages {
        stage('Build Proxy to Spec') {
            agent {
                docker { image 'chronos085/node-apigee:8-alpine' }
            }
            steps {
                sh 'openapi2apigee generateApi proxy -s openapi.yaml -d apigee'
                sh 'apigeelint -s apigee/proxy/apiproxy -f table.js'
                sh 'git checkout -b master'
                sh 'git add apigee'
                sh 'git commit -m "proxy commit"'
                sh 'git update-ref HEAD master'
                sh 'git push -u origin master'
            }
        }
    }
}
