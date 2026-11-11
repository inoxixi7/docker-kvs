// pipeline {
//   agent any
//   environment {
//     DOCKERHUB_USER = "oca2402pg0005"
//     BUILD_HOST = "root@172.22.27.241"
//     PROD_HOST = "root@172.22.27.242"
//     BUILD_TIMESTAMP = sh(script: "date +%Y%m%d-%H%M%S", returnStdout: true).trim()
//   }
//   stages {
//     stage('Pre Check') {
//       steps {
//         sh "test -f ~/.docker/config.json"
//         sh "cat ~/.docker/config.json | grep docker.io"
//       }
//     }
//     stage('Build') {
//       steps {
//         sh "cat docker-compose.build.yml"
//         sh "docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml down"
//         sh "docker -H ssh://${BUILD_HOST} volume prune -f"
//         sh "docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml build"
//         sh "docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml up -d"
//         sh "docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml ps"
//       }
//     }
//     stage('Test') {
//       steps {
// 		sh "cat docker-compose.build.yml"
//         //sh "docker -H ssh://${BUILD_HOST} container exec dockerkvs_apptest pytest -v test_app.py"
//         //sh "docker -H ssh://${BUILD_HOST} container exec dockerkvs_webtest pytest -v test_static.py"
//         //sh "docker -H ssh://${BUILD_HOST} container exec dockerkvs_webtest pytest -v test_selenium.py"
//         //sh "docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml down"
//       }
//     }
//     stage('Register') {
//       steps {
//         sh "docker -H ssh://${BUILD_HOST} tag dockerkvs_web ${DOCKERHUB_USER}/dockerkvs_web:${BUILD_TIMESTAMP}"
//         sh "docker -H ssh://${BUILD_HOST} tag dockerkvs_app ${DOCKERHUB_USER}/dockerkvs_app:${BUILD_TIMESTAMP}"
//         sh "docker -H ssh://${BUILD_HOST} push ${DOCKERHUB_USER}/dockerkvs_web:${BUILD_TIMESTAMP}"
//         sh "docker -H ssh://${BUILD_HOST} push ${DOCKERHUB_USER}/dockerkvs_app:${BUILD_TIMESTAMP}"
//       }
//     }
//     stage('Deploy') {
//       steps {
//         sh "cat docker-compose.prod.yml"
//         sh "echo 'DOCKERHUB_USER=${DOCKERHUB_USER}' > .env"
//         sh "echo 'BUILD_TIMESTAMP=${BUILD_TIMESTAMP}' >> .env"
//         sh "cat .env"
//         sh "docker-compose -H ssh://${PROD_HOST} -f docker-compose.prod.yml up -d"
//         sh "docker-compose -H ssh://${PROD_HOST} -f docker-compose.prod.yml ps"
//       }
//     }
//   }
// }



pipeline {
  agent any

  environment {
    DOCKERHUB_USER = "oca2402pg0005"
    BUILD_HOST = "root@172.22.27.241"
    PROD_HOST = "root@172.22.27.242"
    BUILD_TIMESTAMP = sh(
      script: "date +%Y%m%d-%H%M%S",
      returnStdout: true
    ).trim()
  }

  stages {

    stage('Pre Check') {
      steps {
        sh 'docker --version'
        sh 'docker-compose --version'
        echo "Docker Hub User: ${DOCKERHUB_USER}"
        echo "Build Host: ${BUILD_HOST}"
        echo "Production Host: ${PROD_HOST}"
      }
    }

    stage('Build') {
      steps {
        sh "cat docker-compose.build.yml"

        sh "docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml down"

        sh "docker -H ssh://${BUILD_HOST} volume prune -f"

        sh "docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml build"

        sh "docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml up -d"

        sh "docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml ps"
      }
    }

    stage('Test') {
      steps {
        sh "cat docker-compose.build.yml"

        // 测试通过后再启用
        // sh "docker -H ssh://${BUILD_HOST} container exec dockerkvs_apptest pytest -v test_app.py"
        // sh "docker -H ssh://${BUILD_HOST} container exec dockerkvs_webtest pytest -v test_static.py"
        // sh "docker -H ssh://${BUILD_HOST} container exec dockerkvs_webtest pytest -v test_selenium.py"
      }
    }

    stage('Register') {
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: 'dockerhub-credentials',
            usernameVariable: 'DH_USER',
            passwordVariable: 'DH_TOKEN'
          )
        ]) {
          sh '''
            set +x

            # 登录 Docker Hub
            echo "$DH_TOKEN" | docker login \
              -u "$DH_USER" \
              --password-stdin

            # 给远程 Build Host 的镜像打 Tag
            docker -H ssh://root@172.22.27.241 tag \
              dockerkvs_web \
              ${DOCKERHUB_USER}/dockerkvs_web:${BUILD_TIMESTAMP}

            docker -H ssh://root@172.22.27.241 tag \
              dockerkvs_app \
              ${DOCKERHUB_USER}/dockerkvs_app:${BUILD_TIMESTAMP}

            # 推送 Web 镜像
            docker -H ssh://root@172.22.27.241 push \
              ${DOCKERHUB_USER}/dockerkvs_web:${BUILD_TIMESTAMP}

            # 推送 App 镜像
            docker -H ssh://root@172.22.27.241 push \
              ${DOCKERHUB_USER}/dockerkvs_app:${BUILD_TIMESTAMP}

            # 退出 Docker Hub
            docker logout
          '''
        }
      }
    }

    stage('Deploy') {
      steps {
        sh "cat docker-compose.prod.yml"

        sh "echo 'DOCKERHUB_USER=${DOCKERHUB_USER}' > .env"

        sh "echo 'BUILD_TIMESTAMP=${BUILD_TIMESTAMP}' >> .env"

        sh "cat .env"

        // 先从 Docker Hub 拉取镜像
        sh "docker-compose -H ssh://${PROD_HOST} -f docker-compose.prod.yml pull"

        // 启动生产环境
        sh "docker-compose -H ssh://${PROD_HOST} -f docker-compose.prod.yml up -d"

        sh "docker-compose -H ssh://${PROD_HOST} -f docker-compose.prod.yml ps"
      }
    }
  }

  post {
    success {
      echo "Deployment completed successfully!"
    }

    failure {
      echo "Pipeline failed! Please check Jenkins logs."
    }
  }
}
