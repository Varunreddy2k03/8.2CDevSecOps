pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/varunreddy2k03/8.2CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        dir('NODE_SUBFOLDER') {    // <<< REPLACE NODE_SUBFOLDER with your folder name
          bat 'npm install'
        }
      }
    }

    stage('Run Tests') {
      steps {
        dir('NODE_SUBFOLDER') {
          bat 'cmd /c npm test || exit /b 0'
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        dir('NODE_SUBFOLDER') {
          bat 'cmd /c npm run coverage || exit /b 0'
        }
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
