pipeline {
  agent any
  options { timestamps() }
  environment {
    PYTHON = 'python3'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Set up Python') {
      steps {
        sh 'python -V || true'
        sh 'pip -V || true'
        sh 'python -m venv .venv || true'
        sh '. .venv/bin/activate && python -m pip install --upgrade pip'
        sh '. .venv/bin/activate && pip install -r requirements.txt'
      }
    }
    stage('Lint') {
      steps {
        sh '. .venv/bin/activate && flake8 app tests --max-line-length 100'
      }
    }
    stage('Unit Tests + Coverage + HTML') {
      steps {
        sh 'mkdir -p reports'
        sh '. .venv/bin/activate && pytest -q --cov=app --cov-report=term-missing --html=reports/pytest-report.html --self-contained-html'
      }
      post {
        always { archiveArtifacts artifacts: 'reports/pytest-report.html', fingerprint: true, onlyIfSuccessful: false }
      }
    }
    stage('BDD (Behave)') {
      steps {
        sh '. .venv/bin/activate && behave -f html -o reports/behave-report.html'
      }
      post {
        always { archiveArtifacts artifacts: 'reports/behave-report.html', fingerprint: true, onlyIfSuccessful: false }
      }
    }
    stage('Start Flask (bg)') {
      steps {
        sh 'nohup . .venv/bin/activate && python -m flask --app app.main:app run --port 5000 > flask.log 2>&1 & sleep 3'
      }
      post {
        always { archiveArtifacts artifacts: 'flask.log', fingerprint: true, onlyIfSuccessful: false }
      }
    }
    stage('Locust smoke') {
      steps {
        sh '. .venv/bin/activate && locust -f performance/locustfile.py --headless -u 10 -r 2 -t 15s --csv=reports/locust --host=http://127.0.0.1:5000 --only-summary'
      }
      post {
        always { archiveArtifacts artifacts: 'reports/locust*', fingerprint: true, onlyIfSuccessful: false }
      }
    }
  }
  post {
    always {
      junit allowEmptyResults: true, testResults: '**/junit*.xml'
      archiveArtifacts artifacts: 'reports/**', fingerprint: true, onlyIfSuccessful: false
    }
  }
}
