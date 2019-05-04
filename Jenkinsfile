pipeline {

    options {
        buildDiscarder(logRotator(numToKeepStr: '3', daysToKeepStr: '3'))
    }

    parameters { 
      string(name: 'version', defaultValue: 'r19.5.1', description: 'release version')
      string(name: 'env', defaultValue: 'tst', description: 'deploy env')
    }

    agent any

    stages {
        stage('Clone repos') {
            steps {
                checkout changelog: false, poll: false, 
                  scm: [$class: 'GitSCM', branches: [[name: '*/${version}']], 
                  doGenerateSubmoduleConfigurations: false, 
                  extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'app']],  
                  submoduleCfg: [], userRemoteConfigs: 
                  [[url: 'ssh://vagrant@192.168.100.100/~/gitserver/helloapp.git']]]

                checkout changelog: false, poll: false, 
                  scm: [$class: 'GitSCM', branches: [[name: '*/${env}']], 
                  doGenerateSubmoduleConfigurations: false, 
                  extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'env']],  
                  submoduleCfg: [], userRemoteConfigs: 
                  [[url: 'ssh://vagrant@192.168.100.100/~/gitserver/helloenv.git']]]
            }
        }

        stage('Run playbook') {
            steps {
              ansiblePlaybook disableHostKeyChecking: true, extras: '-e "version=${version}"', inventory: 'hosts', playbook: 'webdeployer.yml'
            }
        }

        stage('Trigger next') {
            steps {
              build job: 'welcome_pre', parameters: [string(name: 'version', value: 'r19.5.1'), string(name: 'env', value: 'tst')], propagate: false
            }
        }

    }
}
