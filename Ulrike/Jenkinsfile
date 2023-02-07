pipeline {
    agent any
    environment {
        ACRCreds = credentials('acr_creds')
       
    }
    
    stages {
        
        stage('ACR Login') {
            steps{
                sh 'docker login devops2022.azurecr.io -u $ACRCreds_USR -p $ACRCreds_PSW'
            }
        }

        stage('Image Building and push ACR') {                   
            steps {
                sh 'docker build -t devops2022.azurecr.io/ullis-image:$GIT_COMMIT .' //image wird auf Jenkins VM gebaut aus Dockerfile (im git Repo) und getagt
                sh 'docker push devops2022.azurecr.io/ullis-image:$GIT_COMMIT'              //image wird auf ACR gepushed
                sh 'docker rmi devops2022.azurecr.io/ullis-image:$GIT_COMMIT'               // image wird von Jenkins VM gelöscht
            }

        }

        stage('Deploy') {     //so dass nachher der immer neue Imagename auf
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '2eb747c4-f19f-4601-ab83-359462e62482',  url: 'https://github.com/Brights-DevOps-2022-Script/mecomTeam.git']]])
                withCredentials([usernamePassword(credentialsId: '2eb747c4-f19f-4601-ab83-359462e62482', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                sh 'sed -i "s|image:.*|image: devops2022.azurecr.io/ullis-image:$GIT_COMMIT|" Ulrike/nginx-deployment.yaml'
                sh "git add Ulrike/nginx-deployment.yaml"
                sh "git commit -m 'update image'"
                sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Brights-DevOps-2022-Script/mecomTeam.git HEAD:main"
                }                            
                // kustomize.yaml ... kustomize edit set image dev....:$hsh
            }

        }

        // stage('commit deployment with new imagename auf ArgoCD-Git')
        // git commit -am "automated update from jenkins $hash"
        // git push


       // stage('deploy to k8s') {                  // das macht jetzt ArgoCD für uns mit dem deployment aus ArgoCD Git
       //     agent {
       //         docker {
       //             image 'alpine/k8s:1.23.16'                                  // in diesem Image laufen die Befehle an K8s, da hier kubectl verstanden werden kann
       //         }
       //     }

       //     environment{
       //         KUBECONFIG = credentials('k8s_config')                          // credentials von unserem k8s Cluster, in Jenkins angelegt
       //     }

       //     steps {
       //         sh 'echo $KUBECONFIG'
       //         sh 'kubectl --kubeconfig=$KUBECONFIG apply -f namespace.yaml'
       //         sh 'sed "s|REPLACE_ME|devops2022.azurecr.io/ullis-image:$GIT_COMMIT|" -i nginx-deployment.yaml' //ersetzt image im deployment-file durch obigen imagenamen
        //        sh 'kubectl --kubeconfig=$KUBECONFIG apply -f nginx-deployment.yaml'            // das zieht das Image aus ACR, drinnen benannt, und baut pods draus.            
        //        sh 'kubectl --kubeconfig=$KUBECONFIG apply -f service.yaml'
        //        sh 'kubectl --kubeconfig=$KUBECONFIG apply -f loadbalancer.yaml'

               


        //    }
        }

}    
    

