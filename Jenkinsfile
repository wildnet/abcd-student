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
				/*sh 'docker start juice-shop || docker run --name juice-shop -d --rm -p 172.17.0.1:3000:3000 -p 127.0.0.1:3000:3000 bkimminich/juice-shop'
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
				}*/
				sh 'mkdir -p results'
				echo 'start env'
				sh 'ls'
            }
		}
		stage('DAST: Active scan [ZAP]') {
			steps {
				/*sh '''
					docker run --name zap \
						--add-host=host.docker.internal:host-gateway \
						-v "/home/michal/abcdso/abcd-student/.zap:/zap/wrk/:rw" \
						-t ghcr.io/zaproxy/zaproxy:stable bash -c \
						"zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/active.yaml" || true
				'''*/
				echo 'DAST [ZAP]'
				sh 'ls'
			}
			post {
				always {
					/*sh '''
						docker cp zap:/zap/wrk/reports/zap_html_report.html "${WORKSPACE}/results/."
						docker cp zap:/zap/wrk/reports/zap_xml_report.xml "${WORKSPACE}/results/."
						docker stop zap juice-shop
						docker rm zap
					'''*/
					echo 'always'
					sh 'ls'
				}
			}
		}
		stage('SCA: [OSV-scanner]') {
			steps {
				/*sh '''
					docker run --name osv-scanner-json \
					-v /home/michal/abcdso/abcd-student/:/app \
					-v /home/michal/abcdso/reports/:/reports:rw \
					-t osv-scanner:latest \
					--lockfile /app/package-lock.json \
					--format json \
					--output /reports/osv-scan_report.json || true
				'''
				sh '''
					docker run --name osv-scanner-txt \
					-v /home/michal/abcdso/abcd-student/:/app \
					-v /home/michal/abcdso/reports/:/reports:rw \
					-t osv-scanner:latest \
					--lockfile /app/package-lock.json \
					--format table \
					--output /reports/osv-scan_report.txt || true
				'''*/
				echo 'SCA: [OSV-scanner]'
				sh 'ls'
			}
			post {
				always {
					/*sh '''
						docker cp osv-scanner-json:/reports/osv-scan_report.json "${WORKSPACE}/results/."
						docker cp osv-scanner-txt:/reports/osv-scan_report.txt "${WORKSPACE}/results/."
						docker rm osv-scanner-json osv-scanner-txt
					'''*/
					echo 'always'
					sh 'ls'
				}
			}
		}
		stage('SAST: [TruffleHog]') {
			steps {
				//sh 'docker run --rm -v "$PWD:/pwd" trufflesecurity/trufflehog:latest github --repo https://github.com/wildnet/abcd-student --only-verified --issue-comments --pr-comments --json > ${WORKSPACE}/results/trufflehog_report.json'
				echo 'SAST: [TruffleHog]'
				sh 'ls'
			}
		}
		stage('SAST: [Semgrep]') {
			steps {
				echo 'SAST: [Semgrep]'
				//sh 'semgrep --help'
				sh 'semgrep scan --config auto'
				sh 'semgrep scan --baseline-commit --config auto'
				sh 'ls'
			}
		}
		stage('DefectDojoPublisher') {
            steps {
				echo 'Sending reports to DefectDojo'
                //withCredentials([string(credentialsId: 'CREDENTIALS_ID', variable: 'API_KEY')]) {
                    //defectDojoPublisher(artifact: 'results/osv-scan_report.json', productName: 'Juice Shop', scanType: 'OSV-Scanner Scan', engagementName: 'michal.lesniewski@opi.org.pl', defectDojoCredentialsId: API_KEY, sourceCodeUri: 'https://git.com/org/project.git', branchTag: 'main')
					//defectDojoPublisher(artifact: 'results/osv-scan_report.json', productName: 'Juice Shop', scanType: 'OSV-Scanner Scan', engagementName: 'michal.lesniewski@opi.org.pl')
				//--defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'michal.lesniewski@opi.org.pl')
				//--defectDojoPublisher(artifact: 'results/osv-scan_report.json', productName: 'Juice Shop', scanType: 'OSV Scan', engagementName: 'michal.lesniewski@opi.org.pl')
				//--defectDojoPublisher(artifact: 'results/trufflehog_report.json', productName: 'Juice Shop', scanType: 'Trufflehog Scan', engagementName: 'michal.lesniewski@opi.org.pl')
				//--defectDojoPublisher(artifact: 'results/semgrep_report.json', productName: 'Juice Shop', scanType: 'Semgrep JSON Report', engagementName: 'michal.lesniewski@opi.org.pl')
                //}
            }
        }
    }
	post {
		always {
			echo 'Archiving reports'
			archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
			//echo 'Sending reports to DefectDojo'
			//defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'michal.lesniewski@opi.org.pl')
			//defectDojoPublisher(artifact: 'results/osv-scan_report.json', productName: 'Juice Shop', scanType: 'OSV Scan', engagementName: 'michal.lesniewski@opi.org.pl')
		}
	}
}
