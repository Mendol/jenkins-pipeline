pipeline {
    agent any
     //环境变量，初始确定后一般不需更改
    tools {
        maven 'mvn3.6'
        jdk   'jdk1.8_master'
    }
    stages {
        stage('拉取源码') {
            steps {
                echo 'Checkout'
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'bc40527b-65c0-48cf-9856-224eb19f2f40', url: 'git@github.com:Mendol/game-of-life.git']]])
            }
        }
        
        stage('静态代码检测/单元测试') {
            steps {
                echo 'Testing'
                //sh 'mvn clean verify sonar:sonar'
                withSonarQubeEnv('SonarQube Server') {
                //注意这里withSonarQubeEnv()中的参数要与之前SonarQube servers中Name的配置相同
                withMaven(maven: 'mvn3.6', options: [jacocoPublisher()]) {
                sh "mvn clean install sonar:sonar -Dsonar.projectKey=gameoflife -Dsonar.projectName=gameoflife001 -Dsonar.projectVersion=1.0 -Dsonar.sourceEncoding=UTF-8 -Dsonar.exclusions=src/test/** -Dsonar.sources=src/ -Dsonar.java.binaries=target/classes -Dmaven.jacoco.reportPaths=target/jacoco.exec -Dsonar.host.url=http://192.168.0.119:9000 -Dsonar.login=ce36c58d80c50b7fb6456c21425a7359b959df10"
                
             }
         }
              script {
               timeout(10) { 
                   //利用sonar webhook功能通知pipeline代码检测结果，未通过质量阈，pipeline将会fail
                   def qg = waitForQualityGate() 
                       if (qg.status != 'OK') {
                           error "未通过Sonarqube的代码质量阈检查，请及时修改！failure: ${qg.status}"
                       }
                   }
               }
            }
        }
        
        stage('打包') { 
            steps { 
                echo "pipeline success!" 
                archive 'target/*.jar' 
            } 
        }
        
        stage('push_uat') { 
        steps {
            timeout(time: 7, unit: 'DAYS') {
            input message: '是否发布到预生产？',ok: 'Yes'
        }
        sh label: '', script: '/shell/deploy_v2.sh uat'
        }
    }

    }
    
    post {
        
        success {
            emailext (
                subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """ 
                <h1>测试结果 - SUCCEED</h1>
                <span class="tip">(本邮件是程序自动下发的，请勿回复，谢谢！！)</span>
                <h2>Soapui Test Summary:</h2>
                <h2>Build Info:</h2>
                <ul>
                  <li><span>项目名称：</span>${env.JOB_NAME}</li>
                  <li><span>构建编号：</span>${env.BUILD_NUMBER}</li>
                  <li><span>构建结果：</span><a href="${env.BUILD_URL}">Check build results </a></li>
                  <li><span>测试报告：</span><a href="${env.BUILD_URL}/testReport">Browse Test Report</a></li>
                  <li><span>测试日志：</span><a href="${env.BUILD_URL}/console">Open Test Log</a></li>
                  
                </ul>
                
                <!--<p>SUCCEED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>-->
                 """,
                to: "wangmengdi025@163.com",
                from: "wangmd@njiuit.com"
            )
        }
        failure {
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """ <h1>测试结果 - FAILED</h1>
  <span class="tip">(本邮件是程序自动下发的，请勿回复，谢谢！！)</span><p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                    <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
                to: "wangmengdi025@163.com",
                from: "wangmd@njiuit.com"
            )
        }
        
        
        //emailext body: ‘$DEFAULT_CONTENT’, subject: ‘$DEFAULT_SUBJECT’, to: ‘wangmengdi025@163.com' 
    }
    
}
