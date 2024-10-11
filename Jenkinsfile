pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Git code checkout') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-token', url: 'https://github.com/wildnet/abcd-student', branch: 'main'
                }
            }
        }
		stage('Starting environment') {
			steps {
				sh 'docker start juice-shop || docker run --name juice-shop -d --rm -p 172.17.0.1:3000:3000 bkimminich/juice-shop'
				timeout(5) {
 		   			waitUntil {
       					script {
							try {
                				def response = httpRequest 'http://172.17.0.1:3000'
                				return (response.status == 200)
            				}
            				catch (exception) {
                 				return false
            				}
         					//def r = sh script: 'wget -q http://172.17.0.1:3000 -O /dev/null', returnStdout: true
         					//return (r == 0);
       					}
    				}
				}
				sh 'mkdir -p results'
            }
		}
		stage('DAST: Active scan [ZAP]') {
			steps {
				sh '''
					docker run --name zap \
						--add-host=host.docker.internal:host-gateway \
						-v "/home/michal/abcdso/abcd-student/.zap:/zap/wrk/:rw" \
						-t ghcr.io/zaproxy/zaproxy:stable bash -c \
						"zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/active.yaml" || true
				'''
			}
			post {
				always {
					sh '''
						docker cp zap:/zap/wrk/reports/zap_html_report.html "${WORKSPACE}/results/."
						docker cp zap:/zap/wrk/reports/zap_xml_report.xml "${WORKSPACE}/results/."
						docker stop zap juice-shop
						docker rm zap
					'''
				}
			}
		}		
    }
}
