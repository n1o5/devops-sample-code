pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Creating virtual environment and installing dependencies...'
                sh '''
                    python3 -m venv venv
                    ./venv/bin/python -m pip install --upgrade pip
                    ./venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh './venv/bin/python -m unittest discover -s .'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                    mkdir -p ${WORKSPACE}/python-app-deploy
                    cp ${WORKSPACE}/app.py ${WORKSPACE}/python-app-deploy/
                    cp ${WORKSPACE}/requirements.txt ${WORKSPACE}/python-app-deploy/
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Running application...'
                sh '''
                    cd ${WORKSPACE}/python-app-deploy

                    if [ -f app.pid ]; then
                        OLD_PID=$(cat app.pid)
                        if kill -0 "$OLD_PID" 2>/dev/null; then
                            kill "$OLD_PID" || true
                        fi
                    fi

                    nohup ${WORKSPACE}/venv/bin/python app.py \
                        > app.log 2>&1 &

                    echo $! > app.pid
                    sleep 3

                    if ! kill -0 "$(cat app.pid)" 2>/dev/null; then
                        cat app.log
                        exit 1
                    fi
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing application...'
                sh '''
                    ${WORKSPACE}/venv/bin/python \
                        ${WORKSPACE}/test_app.py
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }
    }
}