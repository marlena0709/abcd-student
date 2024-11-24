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
                    git credentialsId: 'github-pat', url: 'https://github.com/marlena0709/abcd-student.git', branch: 'main'
                }
            }
        }
        stage('Example') {
            steps {
                echo 'Hello!'
                sh 'ls -la'
            }
        }
          stage('Prepare') {
            steps {
                echo 'Stage Prepare'
                sh 'mkdir -p results/'
            }
        }
          stage('DAST') {
            steps {
                echo 'DAST'
                sh '''
                    docker run --name juice-shop -d \
                    -p 3000:3000 bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap \
                    --add-host=host.docker.internal:host-gateway \
                    -v /mnt/c/Users/marle/abcd-lab-master/abcd-student/.zap:/zap/wrk/:rw \
                    -t ghrc.io/zaproxy/zaproxy:stable \
                    bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                '''
            }
            post{
                always{
                    sh '''
                        docker cp zap:/zap/wrk/zap_html_report.html ${WORKSPACE}/zap_html_report.html
                        docker cp zap:/zap/wrk/zap_xml_report.xml ${WORKSPACE}/zap_xml_report.xml
                        docker stop zap juice-shop
                        docker rm zap                                        
                    '''
                }
            }
        }
    }
    post{
        always{
            echo'Archiving results...'
            archiveArtifacts artifacts: '**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to DefectDojo...'
            defectDojoPublisher(artifact: 'zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'marlenaaptak@gmail.com')
        }
    }
}
