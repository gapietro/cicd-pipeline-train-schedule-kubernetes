pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "gapietro/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                script {
                   def post = new URL("https://192.168.10.104/api/instances").openConnection();
                   def message = '{"zoneId":1,"instance":{"name":"GPUNSTAL","cloud":"Pietro Local VMWare","site":{"id":1},"type":"ts","instanceType":{"code":"ts"},"instanceContext":"dev","layout":{"id":1202,"code":"760ffff8-d86b-4118-8c3f-8de1c70d90e1"},"plan":{"id":116,"code":"container-256","name":"256MB Memory, 3GB Storage"}},"config":{"resourcePoolId":15,"poolProviderType":"kubernetes","customOptions":{"f_tsver":"10"},"createUser":true},"volumes":[{"id":-1,"rootVolume":true,"name":"root","size":3,"sizeId":null,"storageType":null,"datastoreId":12}],"ports":[{"name":"HTTP","port":31443,"lb":"HTTP"}]}'
                   post.setRequestMethod("POST")
                   post.setDoOutput(true)
                   post.setRequestProperty("Content-Type", "application/json")
                   post.setRequestProperty("Authorization", "Bearer e125ccff-6c05-4664-a21a-500f98e693cc")
                   post.getOutputStream().write(message.getBytes("UTF-8"));
                   def postRC = post.getResponseCode();
                   println(postRC);
                   if(postRC.equals(200)) {
                       println(post.getInputStream().getText());
                   } 
                }
            }
        }
    }
}
