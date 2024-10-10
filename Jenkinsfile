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
					docker run --name juice-shop -d --rm -p 127.0.0.1:3000:3000 bkimminich/juice-shop
					sleep 5
				'''
				sh 'echo "${WORKSPACE}"'
				sh 'ls -alh "${WORKSPACE}"'
				sh '''
					docker run --name zap --rm \
					--add-host=host.docker.internal:host-gateway \
					-v "${WORKSPACE}/.zap/passive.yaml:/zap/wrk/passive_scan.yaml:rw" \
					-v "${WORKSPACE}/.zap/reports:/zap/wrk/reports:rw \
					-t ghcr.io/zaproxy/zaproxy:stable bash -c \
					"ls -alh; pwd; ls -alh /zap/wrk/; ls -alh /zap/wrk/passive_scan.yaml"
				'''
			}
			post {
				always {
					sh '''
						docker cp zap:/zap/wrk/reports/zap_html_report.html "${WORKSPACE}/reports/zap_html_report.html"
						docker cp zap:/zap/wrk/reports/zap_xml_report.xml "${WORKSPACE}/reports/zap_xml_report.xml"
					'''
					sh '''
						docker stop zap juice-shop
					'''
				}
			}
		}
    }
}
