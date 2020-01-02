pipeline {
    agent none
    environment {
        org = 'amer-demo16'
        environment = 'test'
        proxy = 'petstore-jks'
        base64 = credentials('apigee-secret-key')
    }
    stages {
        stage('Build Proxy from Spec') {
            agent {
                docker { image 'chronos085/node-apigee:8-alpine' }
            }
            steps {
                notifySlack('STARTED')
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
                    sh "sed -i.bak '/<PreFlow[[:blank:]]name=\"PreFlow\">/,/<\\/PreFlow>/ s/<Request\\/>/<Request><Step><Name>sf-security-oauth<\\/Name><\\/Step><\\/Request>/g' apigee/proxy/apiproxy/proxies/default.xml"
                    sh 'rm -rf apigee/proxy/apiproxy/proxies/default.xml.bak'
                    //RUN MAVEN
                    sh 'rm -rf apigee/proxy/target'
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
                notifySlack('APPROVE')
                timeout(time: 2, unit: 'DAYS') {
                    input 'Do you want to Approve?'
                }
            }
        }
		stage('Deliver for Development') {
			steps {
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]){
                    //DELIVERY TO DEV
                    sh 'git checkout development'
					sh 'git pull . master'
                }
            }
		}
        stage('Deploy Proxy to Environment') {
            agent {
                docker { image 'chronos085/node-apigee:8-alpine' }
            }
            environment{
                stable_revision = sh(script: 'curl -H "Authorization: Basic ${base64}" "https://api.enterprise.apigee.com/v1/organizations/${org}/apis/${proxy}/deployments" | jq -r ".environment[0].revision[0].name"', returnStdout: true).trim()
            }
            steps {
                script {
                    try {
                        withCredentials([usernamePassword(credentialsId: 'apigee', passwordVariable: 'API_PASSWORD', usernameVariable: 'API_USERNAME')]){
                            sh 'apigeetool deployproxy  -u $API_USERNAME -p $API_PASSWORD -o $org  -e $environment -n $proxy -d apigee/proxy/target'
                        }
                    } catch (err) {
                        echo err.getMessage()
                        withCredentials([usernamePassword(credentialsId: 'apigee', passwordVariable: 'API_PASSWORD', usernameVariable: 'API_USERNAME')]){
                            sh 'apigeetool deployExistingRevision  -u $API_USERNAME -p $API_PASSWORD -o $org  -e $environment -n $proxy -r $stable_revision'
                        }
                        notifySlack('UNDEPLOY')
                    }
                }
            }
        }
    }
    post {
       success {
           notifySlack('SUCCESS')
       }
       failure {
           notifySlack('FAILURE')
       }
       unstable {
           notifySlack('UNSTABLE')
       }
       aborted {
           notifySlack('ABORTED')
       }
    }
}

def notifySlack(String buildStatus = 'STARTED') {
    
    def color
    
    if (buildStatus == 'STARTED') {
        color = '#636363'
    } else if (buildStatus == 'APPROVE') {
        color = '#9128e0'
    } else if (buildStatus == 'SUCCESS') {
        color = '#47ec05'
    } else if (buildStatus == 'UNSTABLE') {
        color = '#d5ee0d'
    } else if (buildStatus == 'ABORTED') {
        color = '#000000'
    } else if (buildStatus == 'UNDEPLOY') {
        color = '#ec2805'
    } else {
        color = '#ec2805'
    }
    
    def msg = "${buildStatus}: `${env.JOB_NAME}` #${env.BUILD_NUMBER}:\n${env.BUILD_URL}"
    
    slackSend(color: color, message: msg)
}
