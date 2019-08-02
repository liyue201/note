## Go项目的jenkinsfile自动化部署模板

一共分成4个步骤

- Check:  使用golangci-lint工具进行静态代码检查
- Build： 编译代码成docker镜像
- Push： 推送镜像到docker仓库
- Deploy： 部署，这里是部署到阿里云的k8s集群，相关配置参考阿里云的相关文档。这里部署时候直接删除老的部署，实际生产环境不推荐这样用。


```

pipeline {
    agent any
    environment {
         REP_URL=""
         APP="myapp"
         TAG="v1.0.0"
         K8S_CLUSTER_NODE = "config-hangzhou-elbs-test"
         K8S_DEPLOY_FILE = "deploy.yaml"
         NAMESPACE = "test"
         DEPLOY_NAME = "myapp-v1.0.0"
    }
    stages {
      stage('Check') {
                steps {
                    script {
                        echo 'check';
                        sh "docker run --rm -v ${WORKSPACE}:/go/src/${APP} -w /go/src/${APP} golangci/golangci-lint:v1.17.1 golangci-lint run --disable errcheck --disable deadcode"
                    }
                }
        }
        stage('Build') {
            steps {
                script {
                    echo 'build';
                    def imageName="${REP_URL}/${APP}:${TAG}"
                    sh "docker build -t ${imageName} -f Dockerfile ."
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    echo 'push'
                    def imageName="${REP_URL}/${APP}:${TAG}"
                    sh "docker push ${imageName}"
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                     echo "delete old deployment"
                     try {
                           sh "kubectl --kubeconfig /home/jenkins/.kube/${K8S_CLUSTER_NODE} " +
                                    "delete deploy ${DEPLOY_NAME} -n ${NAMESPACE}"
                      } catch(error) {
                           echo "delete old deployment failed"
                      }
                     echo "deploy"
                     sh "kubectl --kubeconfig /home/jenkins/.kube/${K8S_CLUSTER_NODE} " +
                         "apply -f  ${WORKSPACE}/${K8S_DEPLOY_FILE} --record"

                     sh "sleep 15"
                     echo "The created pods:"
                     sh "kubectl --kubeconfig /home/jenkins/.kube/${K8S_CLUSTER_NODE}  get pod -o wide -n ${NAMESPACE} | grep ${APP}"
                }
            }
        }
	}
}


```
