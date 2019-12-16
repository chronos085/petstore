pipeline {
    agent none
    stages {
        stage('Build Proxy to Spec') {
            agent {
                docker { image 'chronos085/node-apigee:8-alpine' }
            }
            steps {
                sh 'git status'
                sh 'openapi2apigee generateApi proxy -s openapi.yaml -d apigee'
                sh 'apigeelint -s apigee/proxy/apiproxy -f table.js'
                sh 'git add -A apigee'
                sh 'git status'
                sh 'git commit -m "proxy commit"'
                sh 'git status'
                sh 'git push https://chronos085:naruto2022@github.com/chronos085/petstore.git'
            }
        }
    }
}
