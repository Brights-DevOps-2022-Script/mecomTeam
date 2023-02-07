pipeline {
    agent any 
    stages {
        stage('BUILD') {
            steps {
                withDockerRegistry(credentialsId: 'acr_creds', url: 'https://devops2022.azurecr.io/v2/') {
                sh "docker build -t devops2022.azurecr.io/romanginx:$GIT_COMMIT ./roman/app"
                sh "docker push devops2022.azurecr.io/romanginx:$GIT_COMMIT"
                sh "docker rmi devops2022.azurecr.io/romanginx:$GIT_COMMIT"
                }
            }
        }
        stage('TEST') {
            steps {
                 script {
                    def imageTag = "romanginx:$GIT_COMMIT"
                    def acrLoginServer = "devops2022.azurecr.io"
                    def imageExists = sh(script: "set +x curl -fL ${acrLoginServer}/v2/manifests/${imageTag}", returnStatus: true) == 0
                    if (!imageExists) {
                        error("The image ${imageTag} was not found in the registry ${acrLoginServer}")
                    }
                }
            }
        }
        stage('DEPLOY') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/roman-main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '2eb747c4-f19f-4601-ab83-359462e62482',  url: 'https://github.com/Brights-DevOps-2022-Script/mecomTeam.git']]])
                withCredentials([usernamePassword(credentialsId: '2eb747c4-f19f-4601-ab83-359462e62482', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh "sed -i 's|image:.*|image: devops2022.azurecr.io/romanginx:${GIT_COMMIT}|' roman/k8s/nginx.yml"
                    sh "git add roman/k8s/nginx.yml"
                    sh "git commit -m 'update image'"
                    sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Brights-DevOps-2022-Script/mecomTeam.git HEAD:roman-main"
                }
            }
        }
    }
}
