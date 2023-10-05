node {
    try {
        // Stage Build
        stage('Build') {
            checkout scm
            docker.image('python:3.11.5-alpine3.18').inside {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }

        // Stage Test
        stage('Test') {
            agent any
            docker.image('qnib/pytest').inside {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }

        // Stage Manual Approval
        stage('Manual Approval') {
            agent any
            input message: 'Lanjutkan ke tahap Deploy?'
        }

        // Stage Deploy with delay of 1 min
        stage('Deploy') {
            agent any
            environment {
                VOLUME = "${WORKSPACE}/sources:/src"
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(path: env.BUILD_ID) {
                    unstash(name: 'compiled-results')
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} pyinstaller -F add2vals.py"
                }
                sleep 1
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} rm -rf build dist"
                }
            }
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}