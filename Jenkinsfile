pipeline {
    agent any
    triggers {
        upstream(upstreamProjects: "gate-top", threshold: hudson.model.Result.SUCCESS)
    }
    tools { 
        maven 'Maven Current' 
        jdk 'JDK1.8' 
    }
    options {
        disableConcurrentBuilds()
    }
    stages {
        stage ('Build') {
            steps { 
                sh """
                __condasetup="\$(/home/johann/software/miniconda3/bin/conda 'shell.bash' 'hook' 2> /dev/null)"
                eval "\$__condasetup"
                python --version
                envfile=\$WORKSPACE/tmpenv
                rm -rf \$envfile
                python -m venv \$envfile
                export PATH=\$envfile/bin:\$PATH
                pip install -r \$WORKSPACE/python-requirements.txt
                mvn -e clean install
                """
            }
            post {
                always {
                    junit 'target/surefire-reports/**/*.xml'
                    jacoco exclusionPattern: '**/gate/gui/**,**/gate/resources/**'
                    warnings canRunOnFailed: true, canResolveRelativePaths: false, consoleParsers: [[parserName: 'Java Compiler (javac)']], defaultEncoding: 'UTF-8', excludePattern: '**/test/**', failedNewAll: '0', unstableNewAll: '0', useStableBuildAsReference: true
                }
            }
        }
        stage('Document') {
            when{
                expression { currentBuild.currentResult != "FAILED" && currentBuild.changeSets != null && currentBuild.changeSets.size() > 0 }
            }
            steps {
                sh 'mvn -e -DskipTests site'
            }
            post {
                always {
                    findbugs canRunOnFailed: true, excludePattern: '**/gate/resources/**', failedNewAll: '0', pattern: '**/findbugsXml.xml', unstableNewAll: '0', useStableBuildAsReference: true
                }
                success {
                    step([$class: 'JavadocArchiver', javadocDir: 'target/site/apidocs', keepAll: false])
                }
            }
        }
        stage('Deploy') {
            when{
                branch 'main'
                expression { currentBuild.currentResult == "SUCCESS" && currentBuild.changeSets != null && currentBuild.changeSets.size() > 0 }
            }
            steps {
                sh 'mvn -e -Dmaven.test.skip=true source:jar javadoc:jar deploy'
            }
        }
    }
}
