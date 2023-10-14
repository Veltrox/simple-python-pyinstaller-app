node {
    stage('Build') {
        docker.image('python:3.12.0-alpine3.18').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        junit 'test-reports/results.xml'
    }
    stage('Deliver') {
        node {
            checkout scm
        }
        dir('sources') {
            sh 'python -m py_compile add2vals.py calc.py'
            stash(name: 'compiled-results', includes: '*.py*')
        }
        node {
            checkout scm
        }
        dir('sources') {
            sh 'py.test --junit-xml test-reports/results.xml test_calc.py'
        }
        post {
            always {
                junit 'test-reports/results.xml'
            }
        }
        node {
            checkout scm
        }
        dir('sources') {
            sh "docker run --rm -v ${pwd()}/sources:/src cdrx/pyinstaller-linux:python2 'pyinstaller -F add2vals.py'"
        }
        post {
            success {
                archiveArtifacts 'dist/add2vals'
                sh "rm -rf build dist"
            }
        }
    }
}
