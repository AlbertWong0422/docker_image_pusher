pipeline {
  agent any
  environment {
    DEPLOY_DIR = "/var/www/html"  // 部署目录
    DOCKER_IMAGE = "your-web-app:${env.BUILD_ID}"
  }
  stages {
    stage('代码检出') {
      steps {
        checkout scm
      }
    }
    stage('依赖安装') {
      steps {
        sh 'npm install --registry=https://registry.npmmirror.com'
      }
    }
    stage('项目构建') {
      steps {
        sh 'npm run build'
        archiveArtifacts artifacts: 'dist/**', fingerprint: true
      }
    }
    stage('部署到服务器') {
      steps {
        // SSH传输文件到目标服务器
        sshPublisher(
          publishers: [
            sshPublisherDesc(
              configName: 'prod-web-server',
              transfers: [
                sshTransfer(
                  sourceFiles: 'dist/**',
                  removePrefix: 'dist',
                  remoteDirectory: DEPLOY_DIR,
                  execCommand: "echo '部署完成!'"
                )
              ]
            )
          ]
        )
      }
    }
    stage('容器化部署(可选)') {
      steps {
        script {
          // 构建Docker镜像
          docker.build(DOCKER_IMAGE)
          
          // 停止并移除旧容器
          sh "docker stop web-app || true"
          sh "docker rm web-app || true"
          
          // 运行新容器
          docker.run(
            "-d --name web-app -p 80:80 ${DOCKER_IMAGE}"
          )
        }
      }
    }
  }
  post {
    success {
      slackSend channel: '#deploy-notify', 
                message: "✅ 部署成功: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
    }
    failure {
      slackSend channel: '#deploy-notify', 
                message: "❌ 部署失败: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
    }
  }
}
