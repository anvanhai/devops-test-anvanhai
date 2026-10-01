def notify(String msg) {
  withEnv(["MSG=${msg}"]) {
    sh 'curl -s -X POST "https://api.telegram.org/bot${TG_TOKEN}/sendMessage" -d chat_id=${TG_CHAT} --data-urlencode "text=${MSG}"'
  }
}

pipeline {
  agent any
  tools { nodejs 'node20' }
  triggers { pollSCM('* * * * *') }
    environment {
    TG_TOKEN          = credentials('telegram-token-anvanhai')
    TG_CHAT           = credentials('telegram-chat-id-anvanhai')
    VERCEL_TOKEN      = credentials('vercel-token-anvanhai')
    VERCEL_ORG_ID     = credentials('vercel-org-id-anvanhai')
    VERCEL_PROJECT_ID = credentials('vercel-project-id-anvanhai')
  }
  stages {
    stage('Notify Start') {
      steps {
        script { notify("🚀 DEPLOY STARTED\nProject: devops-test\nBranch: main") }
      }
    }
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Install') {
      steps {
        sh 'if [ -f package.json ]; then npm install; else echo "Static site, no dependencies"; fi'
      }
    }
    stage('Build') {
      steps {
        sh 'if [ -f package.json ] && grep -q "\\"build\\"" package.json; then npm run build; else echo "No build script, skip"; fi'
      }
    }
    stage('Deploy') {
      steps {
        script {
          def out = sh(script: 'npx vercel deploy --prod --yes --token=$VERCEL_TOKEN', returnStdout: true).trim()
          env.SITE_URL = out.readLines().last()
        }
      }
    }
  }
  post {
    success {
      script { notify("✅ DEPLOY SUCCESS\nProject: devops-test\nBranch: main\nURL: ${env.SITE_URL}") }
    }
    failure {
      script { notify("❌ DEPLOY FAILED\nProject: devops-test\nBranch: main\nPlease check Jenkins.") }
    }
  }
}