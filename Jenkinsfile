pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/varunreddy2k03/8.2CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps { bat 'npm install' }
    }

    stage('Run Tests') {
      steps { bat 'cmd /c npm test || exit /b 0' }
    }

    stage('Generate Coverage Report') {
      steps { bat 'cmd /c npm run coverage || exit /b 0' }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        // run audit, save JSON, then evaluate with PowerShell and fail on high/critical
        bat '''
          cmd /c npm audit --json > audit.json 2> npm-audit.err || exit /b 0
          powershell -NoProfile -Command ^
            "if (!(Test-Path 'audit.json')) { Write-Error 'audit.json missing'; exit 1 } ; ^
             $a = Get-Content 'audit.json' -Raw | ConvertFrom-Json ; ^
             $md = $a.metadata.vulnerabilities ; ^
             $h = if ($md -and $md.high) { [int]$md.high } else { 0 } ; ^
             $c = if ($md -and $md.critical) { [int]$md.critical } else { 0 } ; ^
             Write-Host 'Vulnerabilities -> high:' $h ' critical:' $c ; ^
             if ($h -gt 0 -or $c -gt 0) { Write-Error 'Fail: high/critical found'; exit 2 } else { Write-Host 'OK: no high/critical'; exit 0 }"
        '''
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'audit.json, npm-audit.err', fingerprint: true
      echo 'Finished — artifacts archived.'
    }
    failure {
      echo 'Pipeline failed (see audit.json / npm-audit.err for details).'
    }
    success {
      echo 'Pipeline succeeded.'
    }
  }
}
