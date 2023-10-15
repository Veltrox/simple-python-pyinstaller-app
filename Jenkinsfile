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
            agent any
            steps {
                script {
                    def userInput = input(
                        id: 'userInput', message: 'Lanjutkan ke tahap Deploy?',
                        parameters: [choice(name: 'Proceed', choices: 'Proceed\nAbort', description: 'Pilih Proceed untuk melanjutkan atau Abort untuk menghentikan pipeline', defaultValue: 'Abort')]
                    )
                    if (userInput == 'Proceed') {
                        currentBuild.result = 'CONTINUE'
                    } else {
                        error('Pipeline dihentikan oleh pengguna')
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
                dir(path: "${env.BUILD_ID}") {
                    unstash(name: 'compiled-results')
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} pyinstaller -F add2vals.py"
                }
                script {
                    currentBuild.result = 'SUCCESS' // Menandai bahwa aplikasi telah berhasil dideploy
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: "${env.BUILD_ID}/sources/dist/add2vals", onlyIfSuccessful: true
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} rm -rf build dist"
                    echo 'Aplikasi berhasil di-deploy. Akan menjalankan selama 1 menit sebelum otomatis berakhir.'
                    script {
                        sleep time: 1, unit: 'MINUTES' // Menunggu selama 1 menit
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
