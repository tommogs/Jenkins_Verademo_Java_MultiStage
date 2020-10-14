pipeline {
    agent {
        docker {
            image 'maven:3.6-jdk-11'
            args '-v /root/.m2:/root/.m2'
        }
    }
    options {
        skipStagesAfterUnstable()
    }
    environment {
        JAVA_OPTS="-Duser.home=${WORKSPACE}"
    }
    
    stages {
        stage('Build') {
            steps {
                // Compile Java app
                sh 'mvn clean package'
                // pull docker container
                //sh 'doker pull juliantotzek/verademo1-tomcat'
            }
        }
        stage('Security Scan Master Branch') {
            when {
                expression {env.GIT_BRANCH == 'origin/master'}    
            }
            steps {
                parallel(
                    a:{
                        // Policy scan
                        withCredentials([usernamePassword(credentialsId: 'VeracodeAPI', passwordVariable: 'VERACODEKEY', usernameVariable: 'VERACODEID')]) {
                            veracode applicationName: "Jenkins_Verademo_Java_MultiStage", canFailJob: true, timeout: 60, criticality: 'VeryHigh',
                            fileNamePattern: '', replacementPattern: '', scanExcludesPattern: '', scanIncludesPattern: '',
                            scanName: 'build $buildnumber - Jenkins',
                            uploadExcludesPattern: '', uploadIncludesPattern: 'target/*.war', waitForScan: true,
                            vid: VERACODEID, vkey: VERACODEKEY
               
                        }
                    },
               //     b:{
                        // 3rd party scan application
               //         catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE')
               //         withCredentials([string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
               //             sh 'curl -sSL https://download.sourceclear.com/ci.sh | sh'
               //         }
               //     },
               //     c:{
                        // 3rd party scan docker container
               //         withCredentials([string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
               //             sh 'curl -sSL https://download.sourceclear.com/ci.sh | sh -s scan --image juliantotzek/verademo1-tomcat'
               //         }
               //     }
                )
            }
        }
        stage('Security Scan Feature Branch'){
            when {
                branch "release"
            }
            steps {
                parallel(
                    a:{
                        // Sandbox scan
                        withCredentials([usernamePassword(credentialsId: 'VeracodeAPI', passwordVariable: 'VERACODEKEY', usernameVariable: 'VERACODEID')]) {
                            veracode applicationName: "Verademo", criticality: 'VeryHigh', createSandbox: true, sandboxName: "jenkins-release", 
                            fileNamePattern: '', replacementPattern: '', scanExcludesPattern: '', scanIncludesPattern: '',
                            scanName: 'build $buildnumber - Jenkins',
                            uploadExcludesPattern: '', uploadIncludesPattern: 'target/*.war',
                            vid: VERACODEID, vkey: VERACODEKEY
                        }
                    },
                  //  b:{
                  //     // 3rd party scan application
                  //      withCredentials([string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
                  //          sh 'curl -sSL https://download.sourceclear.com/ci.sh | sh --debug'
                  //     }
                  //  },
                  //  c:{
                        // 3rd party scan docker container
                  //      withCredentials([string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
                  //          sh 'curl -sSL https://download.sourceclear.com/ci.sh | sh -s scan --image juliantotzek/verademo1-tomcat --debug'
                  //      }
                  //  }
                )
            }
        }
        stage('Security Scan Development Branch'){
            when {
                branch "development"
            }
            steps{
                parallel(
                    a:{
                        //Pipeline scan
                        withCredentials([usernamePassword(credentialsId: 'VeracodeAPI', passwordVariable: 'VERACODEKEY', usernameVariable: 'VERACODEID')]) {
                            sh 'curl -O https://downloads.veracode.com/securityscan/pipeline-scan-LATEST.zip'
                            sh 'unzip -o pipeline-scan-LATEST.zip pipeline-scan.jar'
                            sh '''java -jar pipeline-scan.jar -vid "$VERACODEID" -vkey "$VERACODEKEY" --file target/verademo.war'''
                        }
                    },
                    b:{
                        // 3rd party scan application
                        withCredentials([string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
                            sh 'curl -sSL https://download.sourceclear.com/ci.sh | sh'
                        }
                    },
                    c:{
                        // 3rd party scan docker container
                        withCredentials([string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
                            sh 'curl -sSL https://download.sourceclear.com/ci.sh | sh -s scan --image juliantotzek/verademo1-tomcat'
                        }
                    }
                )
            }
        }
        stage ('Deploy Application into docker and start docker'){
            when {
                anyOf{
                    branch 'master';
                    branch 'release'
                }
            }
            steps{
                // Deploy Application into docker and start docker
                //sh '''docker -H3.120.207.156:2375 run  --detach --network verademo --hostname verademo.verademo.com --network-alias verademo.verademo.com -p 80:8080 --name verademo --restart=no --volume /Users/ubuntu/docker/volumes/verademo/data:/var/verademo_home verademo
                //    docker -H3.120.207.156:2375 start verademo'''
                echo 'do nothing'
            }
        }
        stage ('Security Scan - Dynamic Analysis'){
            when {
                branch 'master'
                branch 'release'
            }
            steps {
                // Dynamic Analysis
                withCredentials([usernamePassword(credentialsId: 'VeracodeAPI', passwordVariable: 'VERACODEKEY', usernameVariable: 'VERACODEID')]) {
                    veracodeDynamicAnalysisResubmit analysisName: 'Dynamic Analysis 24 Jan 2019 11:14:11', maximumDuration: 72, vid: VERACODEID, vkey: VERACODEKEY
                }
            }
        }
    }
}
