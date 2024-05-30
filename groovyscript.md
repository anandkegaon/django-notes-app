pipeline {
    agent any

    stages {
        stage('git chckout') {
            steps {
             git branch: 'main', url: 'https://github.com/anandkegaon/django-notes-app.git'
            }
        }
         stage('build') {
            steps {
                echo "build the build"
                sh "docker build -t noteapp ."
            }
        }
         stage('push') {
            steps {
             echo "push build"
              withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKERHUB_Pass', usernameVariable: 'DOCKERHUB_USR')]) {
                 sh "docker tag noteapp ${env.DOCKERHUB_USR}/noteapp:latest " 
                 sh "docker login -u ${env.DOCKERHUB_USR} -p ${env.DOCKERHUB_Pass}"
                 sh "docker push ${env.DOCKERHUB_USR}/noteapp:latest "
             }
            }
        }
         stage('deploy') {
            steps {
                echo "deploy build"
                sh "docker run -d -p 8000:8000 nandu0725/noteapp:latest"
            }
        }
    }
}
