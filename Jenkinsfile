pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                sh 'git pull origin main'  // Ensures the latest files are fetched
            }
        }

        stage('Build') {
            steps {
                sh '''#!/bin/bash
                python3 -m venv venv
                source venv/bin/activate
                pip install --upgrade pip
                if [ -f requirements.txt ]; then
                    pip install -r requirements.txt
                else
                    echo "ERROR: requirements.txt not found!" >&2
                    exit 1
                fi
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''#!/bin/bash
                source venv/bin/activate
                pytest
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''#!/bin/bash
                source venv/bin/activate
                nohup python app.py > app.log 2>&1 &
                '''
            }
        }
    }
}
