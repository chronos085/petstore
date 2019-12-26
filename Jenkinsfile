pipeline {
    agent none
    stages {
        stage('Build Proxy from Spec') {
            agent {
                docker { image 'chronos085/node-apigee:8-alpine' }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]){
                    //CREATE PROXY
                    sh 'openapi2apigee generateApi proxy -s openapi.yaml -d apigee'
                    sh 'rm -rf apigee/proxy/apiproxy.zip'
                    //TEST CODE PROXY
                    sh 'apigeelint -s apigee/proxy/apiproxy -f table.js'
                    //COMMIT CREATE PROXY
                    sh 'git add -A apigee'
                    sh 'git commit -m "proxy commit"'
                    sh 'git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/chronos085/petstore.git HEAD:master'
                }
            }
        }
        stage('Build Proxy to Maven') {
            agent {
                docker { image 'chronos085/maven-apigee:3-alpine' }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]){
                    sh 'rm -rf ci-apigee'
                    sh 'git clone https://$GIT_USERNAME:$GIT_PASSWORD@github.com/chronos085/ci-apigee.git'
                    //ADD MAVEN
                    sh 'cp -r ci-apigee/maven/pom.xml apigee/proxy'
                    sh 'cp -r ci-apigee/maven/shared-pom.xml apigee/proxy'
                    //ADD CONFIG
                    sh 'cp -r configs/config.json apigee/proxy'
                    //ADD POLICIES
                    sh 'mkdir -p apigee/proxy/apiproxy/policies'
                    sh 'cp -r ci-apigee/templates/sharedflows/security-oauth/sf-security-oauth.xml apigee/proxy/apiproxy/policies'
                    //ADD POLICIES XML
                    sh 'sed "/<PreFlow name="PreFlow">/,/<\/PreFlow>/ s/<Request\/>/<Request><Step><Name>sf-security-oauth<\/Name><\/Step><\/Request>/g;" apigee/proxy/apiproxy/proxies/default.xml'
                    //RUN MAVEN
                    sh 'mkdir -p apigee/proxy/target'
                    sh 'mkdir -p apigee/proxy/target/apiproxy'
                    sh 'mvn -f apigee/proxy/pom.xml package -Pbuild -Doptions=inactive'
                    //COMMIT MAVEN PROXY
                    sh 'git pull https://$GIT_USERNAME:$GIT_PASSWORD@github.com/chronos085/petstore.git HEAD:master'
                    sh 'git add -A apigee'
                    sh 'git commit -m "proxy commit"'
                    sh 'git tag -a petstore-$BUILD_NUMBER -m "Jenkins"'
                    sh 'git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/chronos085/petstore.git HEAD:master  --tags'
                }
            }
        }
        stage('Promotion') {
            steps {
                timeout(time: 2, unit: 'DAYS') {
                    input 'Do you want to Approve?'
                }
            }
        }
        stage('Deploy Proxy to Enviroment') {
            agent {
                docker { image 'chronos085/node-apigee:8-alpine' }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'apigee', passwordVariable: 'API_PASSWORD', usernameVariable: 'API_USERNAME')]){
                    sh 'apigeetool deployproxy  -u $API_USERNAME -p $API_PASSWORD -o amer-demo16  -e test -n petstore-$BUILD_NUMBER -d apigee/proxy/target'
                }
            }
        }
    }
}
