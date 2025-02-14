pipeline {
    agent any
    environment {
        // DEPLOYMENT_NAME = 'elis'
        CHROME_BIN= '/usr/bin/google-chrome'
        environment = 	'crm.elis.ro'
    }
    parameters {
    string(defaultValue: 'Jenkinsfile', description: '', name: 'Version', trim: true)
    }
    stages {
        stage("cleanws"){
            steps{
                cleanWs()
            }
        }
        stage("git"){
        parallel {
            stage('Git Backoffice') {
                steps { 
                    
                    git credentialsId: 'gitkey', branch:'develop', url: 'git@bitbucket.org:ddcmteam/backoffice.git'
                }
            }
            stage('Git angular-ddcm') {
                steps { 
                        dir('src/app/modules/angular-ddcm') {
                    
                        git credentialsId: 'gitkey',branch:'develop', url: 'git@bitbucket.org:ddcmteam/angular-ddcm.git'
                    }
                }
            }
            stage('Git elis') {
                steps { 
                        dir('src/app/modules/angular-ddcm/forms-ddcm/components/elis') {
                        git credentialsId: 'gitkey',branch:'master', url: 'git@bitbucket.org:ddcmteam/elis-forms-ddcm.git'
                    }
                }
            }
            stage('Git asigno') {
                steps { 
                        dir('src/app/modules/angular-ddcm/forms-ddcm/components/asigno') {
                        git credentialsId: 'gitkey', url: 'git@bitbucket.org:ddcmteam/asigno-forms-ddcm.git'
                    }
                }
            }
            stage('Git devops') {
                steps { 
                    dir('devops') {
                        checkout scmGit(branches: [[name: '*/refreshStack']], extensions: [cloneOption(depth: 3, honorRefspec: true, noTags: true, reference: '', shallow: true)], userRemoteConfigs: [[credentialsId: 'gitkey', url: 'git@bitbucket.org:ddcmteam/devops.git']])
                    }
                }
            }
        }

            
        }
        stage('Build Backoffice'){
            steps {
                sh 'bash -l -c ". $HOME/.nvm/nvm.sh ; nvm use 12.22.1 || nvm install 12.22.1 && nvm use 12.22.1; node updateEnvironment.js url ${environment} ;npm run enable-apm; npm install; npm run setup-elis; npm run build:prod --theme=elis"' 
            }
        }
        stage('Create tar'){
            steps {
                sh "tar -zcvf artifacts.tar.gz dist/"
            }
        }
        stage('Preparing tar for building'){
            steps {
                sh "mv artifacts.tar.gz devops/CI/Nginx-Frontend/artifacts.tar.gz"
            }
        }
        stage('Build and publish image'){
            steps {
                dir('devops/CI/Nginx-Frontend'){
                    sh "sudo podman build . -t artifacts.asigno.ro/ddcm/backoffice:${Version} --storage-opt ignore_chown_errors=true"
                          withCredentials([usernamePassword(credentialsId: 'NexusRepo', passwordVariable: 'pass', usernameVariable: 'user')]) {
                            sh "sudo podman login -u=$user -p=$pass artifacts.asigno.ro"
                        }
                    sh "sudo podman push artifacts.asigno.ro/ddcm/backoffice:${Version}"
                    sh "sudo podman image rm artifacts.asigno.ro/ddcm/backoffice:${Version} -f"
                }
            }
        }
    }
}
