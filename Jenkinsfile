pipeline {
  agent {
    node {
        kubernetes {
        label 'ruby'
        yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jekyll/jekyll
    image: jekyll/jekyll:latest
    tty: true
"""
        }
    }

  }
  environment {
    REPOSITIRY = 'git@github.com:frowhy/quarkus.asia.github.io.git'
    REGISTRY = 'frowhy'
    APP_NAME = 'quarkus-asia'
    DOCKERHUB_CREDENTIAL = credentials('dockerhub-artifacts')
  }
  stages {
    stage('assets ready') {
      parallel {
        stage('dockerhub login') {
          steps {
            container('ruby') {
              echo '正在登录dockerhub'
              sh 'echo ${DOCKERHUB_CREDENTIAL_PSW} | docker login -u ${DOCKERHUB_CREDENTIAL_USR} --password-stdin'
              echo 'dockerhub登录完毕'
            }
    
          }
        }
        stage('source pull') {
          steps {
            container('ruby') {
              echo '正在拉取源码'
              git(credentialsId: 'deploy-ssh', url: '${REPOSITIRY}', branch: 'master', changelog: true, poll: false)
              echo '源码拉取完毕'
            }
    
          }
        }
        stage('gem configure') {
          steps {
            container('ruby') {
              echo '正在配置gem'
              sh 'gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/'
              echo 'gem配置完毕'
            }
    
          }
        }
      }
    }
    stage('dependencies install') {
      steps {
        container('ruby') {
          echo '正在安装依赖'
          sh 'gem install bundler'
          sh 'bundle install'
          echo '依赖安装完毕'
        }

      }
    }
    stage('assets build') {
      steps {
        container('ruby') {
          echo '正在编译资产'
          sh 'bundle exec jekyll build'
          echo '资产编译完毕'
        }

      }
    }
    stage('image build') {
      steps {
        container('ruby') {
          echo '正在构建镜像'
          sh 'docker build -t ${REGISTRY}/${APP_NAME}:${BUILD_NUMBER} .'
          sh 'docker tag ${REGISTRY}/${APP_NAME}:${BUILD_NUMBER} ${REGISTRY}/${APP_NAME}:latest'
          echo '镜像构建完毕'
        }

      }
    }
    stage('image push') {
      parallel {
        stage('image push') {
          steps {
            container('ruby') {
              echo '正在推送镜像'
              sh 'docker push ${REGISTRY}/${APP_NAME}:${BUILD_NUMBER}'
              echo '镜像推送完毕'
            }

          }
        }
        stage('image latest') {
          steps {
            container('ruby') {
              sh 'docker push ${REGISTRY}/${APP_NAME}:latest'
            }

          }
        }
      }
    }
  }
}