pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-token', url: 'https://github.com/wildnet/abcd-student', branch: 'main'
                }
            }
        }
        stage('Example') {
            steps {
				sh 'pwd'
                sh 'ls -la'
            }
        }
		stage('[ZAP] Baseline passive-scan') {
			steps {
				sh '''
					docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
					sleep 5
				'''
				sh '''
					docker run --name zap -d --rm \\
						--add-host=host.docker.internal:host-gateway \\
						-v "/home/michal/abcdso/abcd-student/.zap:/zap/wrk/:rw" \\
						-t ghcr.io/zaproxy/zaproxy:stable bash -c \\
						"zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
				'''
			}
			post {
				always {
					sh '''
						docker stop zap juice-shop
					'''
				}
			}
		}		
    }
}
