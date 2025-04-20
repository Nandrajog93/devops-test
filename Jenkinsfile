@Library('Shared') _
pipeline {
    agent {label 'Node'}
    
    environment{
        SONAR_HOME = tool "Sonar"
    }
    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (params.FRONTEND_DOCKER_TAG == '') {
                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }
       stage("Workspace cleanup"){
            steps{     
                script{
                    cleanWs()
                }
            }
        }
        stage('Git: Code Checkout') {
            steps {
                script{
                    code_checkout("https://github.com/Nandrajog93/devops-test.git","main")
                }
            }
        }
        stage("Trivy: Filesystem scan"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }

        stage("OWASP: Dependency check"){
            steps{
                script{
                    owasp_dependency()
                }
            }
        }
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("Sonar","devops-test","devops-test")
                }
            }
        }
        stage("SonarQube: Code Quality Gates"){
            steps{
                script{
                    sonarqube_code_quality()
                }
            }
        }
        stage("Docker: Build Images"){
            steps{
                script{
                    
                        dir('frontend'){
                            docker_build("frontend.py","${params.FRONTEND_DOCKER_TAG}","nandrajog93")
                        }
                }
            }
        }
        stage("Docker: Push to DockerHub"){
            steps{
                script{
                    docker_push("frontend","${params.FRONTEND_DOCKER_TAG}","nandrajog93")
                }
            }
        }
    }
    post{
        success{
            archiveArtifacts artifacts: '*.xml', followSymlinks: false
            build job: "Devops-test", parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
                
            ]
        }
    }
}