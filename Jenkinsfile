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
                echo 'Hello!'
                sh 'ls -la'
            }
        }
		stage('[ZAP] Baseline passive-scan') {
			steps {
				sh '''
					docker run --name juice-shop -d --rm -p 127.0.0.1:3000:3000 bkimminich/juice-shop
					sleep 5
				'''
				sh '''
					docker run --name zap --rm --add-host=host.docker.internal:host-gateway -v "${WORKSPACE}:/zap/wrk/:rw" -t ghcr.io/zaproxy/zaproxy:stable bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/.zap/passive.yaml" || true
				'''
				sh '''
					docker run --name zap --rm --add-host=host.docker.internal:host-gateway -v "${WORKSPACE}:/zap/wrk/:rw" -t ghcr.io/zaproxy/zaproxy:stable bash -c "pwd & ls -al /zap/wrk/" || true
				'''
			}
			post {
				always {
					sh '''
						docker cp zap:/zap/wrk/reports/zap_html_report.html "${WORKSPACE}/reports/zap_html_report.html"
						docker cp zap:/zap/wrk/reports/zap_xml_report.xml "${WORKSPACE}/reports/zap_xml_report.xml"
						docker stop zap juice-shop
					'''
				}
			}
		}
    }
}
