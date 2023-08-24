    pipeline {
        agent any
        tools {
            maven "MAVEN3"
            jdk "OracleJDK8"
        }
        environment {
            SNAP_REPO = 'vprofile-snapshot'
            RELEASE_REPO = 'vprofile-release'
            NEXUS_USER = 'admin'
            NEXUS_PASS = 'Pinspire@3042'
            CENTRAL_REPO = 'vpro-maven-central'
            NEXUSIP = '54.158.217.189'
            NEXUSPORT = '8081'
            NEXUS_GRP_REPO = 'vpro-maven-group'
            NEXUS_LOGIN = 'nexuslogin'
            SONARSERVER = 'sonarserver'
            SONARSCANNER = 'sonarscanner'
        }

        stages {
            stage('Build') {
                steps {
                    sh 'mvn -s settings.xml -DskipTests install'
                }
                post {
                    success {
                        echo "Now Archiving.."
                        archiveArtifacts artifacts: '**/*.war'
                    }
                }

            }
            stage('Test') {
                steps {
                    sh 'mvn test'
                }
            }
            stage('Checkstyle Analysis') {
                steps {
                    sh 'mvn -s settings.xml checkstyle:checkstyle'
                }
            }

            stage('Code Analysis With SonarQube') {
                environment {
                    scannerHome = tool "${SONARSCANNER}"
                }
                steps {
                    withSonarQubeEnv("${SONARSERVER}") {
                        sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                                -Dsonar.projectName=vprofile-repo \
                                -Dsonar.projectVersion=1.0 \
                                -Dsonar.sources=src/ \
                                -Dsonar.java.binaries=target \
                                -Dsonar.junit.reportsPath=target/surefire-reports/ \
                                -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                                -Dsonar.java.checkstyle.reportPaths=target/checkstyleresult.xml'''
                    }
                    timeout(time: 10, unit: 'MINUTE') {
                        waitForQUalityGate abortPipeline: true
                    }
                }
            }
        }
    }