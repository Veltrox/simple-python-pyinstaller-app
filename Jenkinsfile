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
        def VOLUME = "${pwd()}/sources:/src"
        def IMAGE = 'cdrx/pyinstaller-linux:python2'
        dir("${env.BUILD_ID}") {
            unstash 'compiled-results'
            sh "docker run --rm -v ${VOLUME} ${IMAGE} pyinstaller -F add2vals.py"
        }
        success {
            archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
            sh "docker run --rm -v ${VOLUME} ${IMAGE} rm -rf build dist"
        }
    }
}
