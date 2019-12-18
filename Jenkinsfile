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
                    sh 'cp -r ci-apigee/maven/pom.xml apigee/proxy'
                    sh 'cp -r ci-apigee/maven/shared-pom.xml apigee/proxy'
                    sh 'cp -r configs/config.json apigee/proxy'
                    sh 'mkdir -p apigee/proxy/target'
                    sh 'mkdir -p apigee/proxy/target/apiproxy'
                    sh 'mvn -f apigee/proxy/pom.xml package -Pbuild -Doptions=inactive'
                    //sh 'git add -A apigee'
                    //sh 'git commit -m "proxy commit"'
                    //sh 'git tag -a petstore-$BUILD_NUMBER -m "Jenkins"'
                    //sh 'git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/chronos085/petstore.git HEAD:master  --tags'
                }
            }
        }
    }
}
