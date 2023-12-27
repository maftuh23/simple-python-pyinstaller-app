node {

    stage('Build') {

        checkout scm

        // Use withDockerContainer to specify the Python container with a custom entrypoint

        withDockerContainer(image: 'python:2-alpine', args: '--entrypoint=""') {

            sh 'python -m py_compile sources/add2vals.py sources/calc.py'

        }

    }

}

node {

    stage('Test') {

        checkout scm

        // Use withDockerContainer to specify the pytest container with a custom entrypoint

        withDockerContainer(image: 'qnib/pytest', args: '--entrypoint=""') {

            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'

            junit 'test-reports/results.xml'

        }

    }

}

node {
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

        }

        post {

            success {

                archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 

                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"

            }

        }
    }
}

node {

    stage('Manual Approval')  {         

    checkout scm         

    // Menunggu input persetujuan dari pengguna         

    input message: 'Lanjutkan ke tahap Deploy?', ok: 'Lanjutkan'     

}

}

node {

    stage('Deploy') {

        sh './jenkins/scripts/deliver.sh' 

        checkout scm

        // Use withDockerContainer to specify the pyinstaller container with a custom entrypoint

        withDockerContainer(image: 'cdrx/pyinstaller-linux:python2', args: '--entrypoint=""') {

            sh 'pyinstaller --onefile sources/add2vals.py'

            archiveArtifacts artifacts: 'dist/add2vals', allowEmptyArchive: true

        }

        echo 'Sleep for 1 min'

        sleep time: 60, unit: 'SECONDS'

    }

}