
pipeline {
    agent any

    environment {
        APP_DIR = "${WORKSPACE}/python-app-deploy"
        VENV = "${WORKSPACE}/.venv"
    }

    stages {

        stage('Build') {
            steps {
                echo 'Installing Python dependencies...'
                sh '''
                    python3 -m venv "$VENV"
                    "$VENV/bin/pip" install --upgrade pip
                    "$VENV/bin/pip" install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    "$VENV/bin/python" -m unittest discover -s . -v
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Python application...'
                sh '''
                    mkdir -p "$APP_DIR"
                    cp app.py requirements.txt "$APP_DIR/"
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting Flask application...'
                sh '''
                    nohup "$VENV/bin/python" "$APP_DIR/app.py" \
                        > "$APP_DIR/app.log" 2>&1 &

                    echo $! > "$APP_DIR/app.pid"
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing running Flask application...'
                sh '''
                    "$VENV/bin/python" -c '
import time
import urllib.request

url = "http://127.0.0.1:5000/"

for attempt in range(15):
    try:
        response = urllib.request.urlopen(url, timeout=2)
        message = response.read().decode()

        print("HTTP Status:", response.status)
        print("Response:", message)

        assert response.status == 200
        assert message == "Hello, Jenkins Multi-Stage Pipeline!"

        print("Application test successful!")
        break

    except Exception:
        if attempt == 14:
            raise
        time.sleep(1)
'
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the logs.'
        }
    }
}
