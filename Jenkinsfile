node {
    // Stage 1: Checkout kode dari repositori
    stage('Checkout') {
        checkout scm
    }
}

// Stage 2: Build
node {
    docker.image('python:3.12.0-alpine3.18').inside {
        sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        stash(name: 'compiled-results', includes: 'sources/*.py*')
    }
}

// Stage 3: Test
node {
    docker.image('qnib/pytest').inside {
        sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
    }
    post {
        always {
            junit 'test-reports/results.xml'
        }
    }
}

// Stage 4: Deliver
node {
    env.VOLUME = sh(script: 'pwd', returnStatus: true).trim() + '/sources:/src'
    env.IMAGE = 'cdrx/pyinstaller-linux:python2'

    dir(path: env.BUILD_ID) {
        unstash(name: 'compiled-results')
        sh "docker run --rm -v ${env.VOLUME} ${env.IMAGE} 'pyinstaller -F add2vals.py'"
    }

    post {
        success {
            archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
            sh "docker run --rm -v ${env.VOLUME} ${env.IMAGE} 'rm -rf build dist'"
        }
    }
}
