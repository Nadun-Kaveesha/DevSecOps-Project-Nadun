pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Nadun-Kaveesha/DevSecOps-Project-Nadun.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=70703628e95224e65a359db801ce193d -t netflix ."
                       sh "docker tag netflix nadun2005/netflix:V${BUILD_NUMBER} "
                       sh "docker push nadun2005/netflix:V${BUILD_NUMBER} "
                    }
                }
            }
        }
        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "DevSecOps-Project-Nadun"
                GIT_USER_NAME = "Nadun-Kaveesha"
            }
            steps {
                withCredentials([string(credentialsId: 'Githubtoken', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "nadunkaveesha2018@gmail.com"
                        git config user.name "Nadun-Kaveesha"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        sed -i "s|image: nadun2005/netflix:latest|image: nadun2005/netflix:V${BUILD_NUMBER}|g" helm/templates/deployment.yml
                        git add .
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                    '''
                }
            }
        }
    }
}
