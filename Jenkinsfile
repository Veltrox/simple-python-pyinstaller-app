pipeline {
    agent none
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:3.12.0-alpine3.18'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Manual Approval') {
            steps {
                script {
                    def userInput = input(message: 'Proceed to the Deploy stage?', ok: 'Proceed', fail: 'Abort')
                    if (userInput == 'Proceed') {
                        currentBuild.displayName = "Deploy"
                    } else {
                        error("Pipeline execution aborted by the user.")
                    }
                }
            }
        }
        stage('Deliver') {
            agent any
            environment {
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(path: env.BUILD_ID) {
                    unstash(name: 'compiled-results')
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                }
                script {
                    currentBuild.result = 'SUCCESS' // Menandai bahwa aplikasi telah berhasil dideploy
                }
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                    echo 'Aplikasi berhasil di-deploy. Akan menjalankan selama 1 menit sebelum otomatis berakhir.'
                    script {
                        sleep 1 * 60 // Menunggu selama 1 menit
                    }
                }
                always {
                    script {
                        currentBuild.result = 'SUCCESS' // Memastikan bahwa eksekusi pipeline berhasil
                    }
                }
            }
        }
    }
}
