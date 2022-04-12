
pipeline 
{
    agent any

    tools 
    {
        maven "MAVEN_HOME"
        //maven "null"
    }
    
    environment {
        DEPLOYMENT_USER     = credentials('deploy_user')
    }

    stages {
        stage('Build') {
            steps {
                //git 'https://github.com/jglick/simple-maven-project-with-tests.git'
                sh "pwd"
                git branch: 'main', url: 'https://github.com/jenkins-test-lab/test.git'
                dir('AnsibleRepo') {
                    git url: 'https://github.com/jenkins-test-lab/Ansitest.git'
                }
                sh "python3 AnsibleRepo/replacetoken.py AnsibleRepo/inventory.ini"
                
                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }

            post {
                success {
                    //junit '**/target/surefire-reports/TEST-*.xml'
                    //archiveArtifacts 'target/*.jar'
                    archiveArtifacts 'multi3/target/*.war'
                }
            }
        }
        
        stage('Static Code Analysis') {
            steps {
                //git 'https://github.com/jglick/simple-maven-project-with-tests.git'
                sh "mvn pmd:pmd"
                sh "mvn checkstyle:checkstyle"
                sh "mvn findbugs:findbugs"
                
                
            }
        }
        
        stage('DeployTestEnv') {
            steps {
                //git 'https://github.com/jglick/simple-maven-project-with-tests.git'
                sh "pwd"
                
                //ansiblePlaybook disableHostKeyChecking: true,playbook: 'AnsibleRepo/main.yml',inventory: 'AnsibleRepo/inventory.ini'
                
                // Run Maven on a Unix agent.
                //deploy adapters: [tomcat9(credentialsId: 'tomcatuser', path: '', url: 'http://20.224.18.124:8080/')], contextPath: 'test', onFailure: false, war: '**/multi3*.war'
                deploy adapters: [tomcat9(credentialsId: 'tomcatuser', path: '', url: 'http://10.1.0.5:8080/')], contextPath: 'test', war: '**/multi3*.war'
            }
        }
        
        stage('AppTest') {
            steps {
                //git 'https://github.com/jglick/simple-maven-project-with-tests.git'
                sh "pwd"
                
                ansiblePlaybook disableHostKeyChecking: true,playbook: 'AnsibleRepo/AppTest.yml',inventory: 'AnsibleRepo/inventory.ini'
                
                
            }
            post {
                success {
                    //junit '**/target/surefire-reports/TEST-*.xml'
                    //archiveArtifacts 'target/*.jar'
                    publishHTML (target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'AnsibleRepo',
                        reportFiles: 'test.html',
                        reportName: "RCov Report"
                    ])
                }
            }
        }
        
        stage('Prod') {
            input{
                message "Do you want to proceed for production deployment?"
            }
            steps {
                //git 'https://github.com/jglick/simple-maven-project-with-tests.git'
                sh "pwd"
                sh "ssh-keygen -R 10.1.0.4"
                
                ansiblePlaybook disableHostKeyChecking: true,playbook: 'AnsibleRepo/ProdDeployment.yml',inventory: 'AnsibleRepo/inventory.ini'
                
                deploy adapters: [tomcat9(credentialsId: 'tomcatuser', path: '', url: 'http://10.1.0.4:8080/')], contextPath: 'prod', war: '**/multi3*.war'
            }
        }
    }
}
