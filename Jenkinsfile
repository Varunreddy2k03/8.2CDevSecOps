pipeline {
  agent any

  environment {
    // change this to your email (comma-separated if multiple)
    EMAIL_RECIPIENTS = 'your.email@example.com'
  }

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
      // archive artifacts so they're preserved in the build and also attachable
      archiveArtifacts artifacts: 'audit.json, npm-audit.err', fingerprint: true
      echo 'Finished — artifacts archived.'

      // send summary email with attachments
      script {
        // currentBuild.currentResult will be set to SUCCESS/FAILURE/etc.
        def status = currentBuild.currentResult ?: 'SUCCESS'
        emailext(
          subject: "${status}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
          body: """<p><b>Job</b>: ${env.JOB_NAME} #${env.BUILD_NUMBER}</p>
                   <p><b>Status</b>: ${status}</p>
                   <p><b>Build URL</b>: <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>
                   <p>Attached: <code>audit.json</code> and <code>npm-audit.err</code> (if present).</p>""",
          to: "${EMAIL_RECIPIENTS}",
          attachmentsPattern: 'audit.json,npm-audit.err',
          mimeType: 'text/html'
        )
      }
    }

    failure {
      echo 'Pipeline failed (see audit.json / npm-audit.err for details).'
    }
    success {
      echo 'Pipeline succeeded.'
    }
  }

}
