pipeline {
    agent none

    triggers {
        pollSCM 'H/10 * * * *'
    }

    options {
        disableConcurrentBuilds()
    }

    stages {
        stage("test: jdk8") {
            agent {
                docker {
                    image 'adoptopenjdk/openjdk8:latest'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh "./mvnw clean dependency:list test -Dsort -B"
            }
        }
        stage("Test") {
            parallel {
//                stage("test: spring-next jdk8") {
//                    agent {
//                        docker {
//                            image 'adoptopenjdk/openjdk8:latest'
//                            args '-v $HOME/.m2:/root/.m2'
//                        }
//                    }
//                    steps {
//                        sh "./mvnw -Pspring-next clean dependency:list test -Dsort -B"
//                    }
//                }
                stage("test: jdk11") {
                    agent {
                        docker {
                            image 'adoptopenjdk/openjdk11:latest'
                            args '-v $HOME/.m2:/root/.m2'
                        }
                    }
                    steps {
                        sh "./mvnw clean dependency:list test -Dsort -B"
                    }
                }
//                stage("test: spring-next jdk11") {
//                    agent {
//                        docker {
//                            image 'adoptopenjdk/openjdk11:latest'
//                            args '-v $HOME/.m2:/root/.m2'
//                        }
//                    }
//                    steps {
//                        sh "./mvnw -Pspring-next clean dependency:list test -Dsort -B"
//                    }
//                }
                stage("test: jdk12") {
                    agent {
                        docker {
                            image 'adoptopenjdk/openjdk12:latest'
                            args '-v $HOME/.m2:/root/.m2'
                        }
                    }
                    steps {
                        sh "./mvnw clean dependency:list test -Dsort -B"
                    }
                }
//                stage("test: spring-next jdk12") {
//                    agent {
//                        docker {
//                            image 'adoptopenjdk/openjdk12:latest'
//                            args '-v $HOME/.m2:/root/.m2'
//                        }
//                    }
//                    steps {
//                        sh "./mvnw -Pspring-next clean dependency:list test -Dsort -B"
//                    }
//                }
            }
        }
        stage('Release to artifactory') {
            when {
                branch 'issue/*'
            }
            agent {
                docker {
                    image 'springci/spring-hateoas-8-jdk-with-graphviz'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            environment {
                ARTIFACTORY = credentials('02bd1690-b54f-4c9f-819d-a77cb7a9822c')
            }

            steps {
                sh "USERNAME=${ARTIFACTORY_USR} PASSWORD=${ARTIFACTORY_PSW} ./mvnw -Pci,snapshot -Dmaven.test.skip=true clean deploy -B"
            }
        }
        stage('Release to artifactory with docs') {
            when {
                branch 'master'
            }
            agent {
                docker {
                    image 'springci/spring-hateoas-8-jdk-with-graphviz'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            environment {
                ARTIFACTORY = credentials('02bd1690-b54f-4c9f-819d-a77cb7a9822c')
                DOC = credentials('02bd1690-b54f-4c9f-819d-a77cb7a9822c')
            }

            steps {
                sh "USERNAME=${ARTIFACTORY_USR} PASSWORD=${ARTIFACTORY_PSW} DOC_USERNAME=${DOC_USR} DOC_PASSWORD=${DOC_PSW} ./mvnw -Pci,snapshot -Dmaven.test.skip=true clean deploy -B"
            }
        }
    }

    post {
        changed {
            script {
                slackSend(
                        color: (currentBuild.currentResult == 'SUCCESS') ? 'good' : 'danger',
                        channel: '#spring-hateoas',
                        message: "${currentBuild.fullDisplayName} - `${currentBuild.currentResult}`\n${env.BUILD_URL}")
                emailext(
                        subject: "[${currentBuild.fullDisplayName}] ${currentBuild.currentResult}",
                        mimeType: 'text/html',
                        recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                        body: "<a href=\"${env.BUILD_URL}\">${currentBuild.fullDisplayName} is reported as ${currentBuild.currentResult}</a>")
            }
        }
    }
}
