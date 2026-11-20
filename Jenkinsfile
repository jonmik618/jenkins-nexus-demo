
pipeline {
    agent any

    environment {
        PATH = "/Users/jon.mikula/.rd/bin:/opt/homebrew/bin:/opt/homebrew/sbin:/usr/local/bin:${env.PATH}"

        IQ_APPLICATION = 'jenkins-nexus-demo__jonmik618'
        IQ_SERVER = 'local-nexus-iq'
    }

    options {
        timestamps()
        skipDefaultCheckout(true)
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Verify Build Environment') {
            steps {
                sh '''
                    echo "=== Java ==="
                    java -version

                    echo "=== Maven ==="
                    mvn -version

                    echo "=== Git ==="
                    git --version

                    echo "=== Project ==="
                    test -f pom.xml
                    echo "pom.xml found"
                '''
            }
        }

        stage('Build and Test') {
            steps {
                sh '''
                    mvn -B clean package
                '''
            }
        }

        stage('Collect Dependencies') {
            steps {
                sh '''
                    mvn -B dependency:copy-dependencies \
                      -DoutputDirectory=target/dependency

                    echo "=== JARs to evaluate ==="
                    find target -name "*.jar" -type f
                '''
            }
        }


        stage('Nexus IQ Policy Evaluation') {
            steps {
                nexusPolicyEvaluation(
                    iqApplication: selectedApplication(
                       'jenkins-nexus-demo__jonmik618'
                    ),
                    iqInstanceId: 'local-nexus-iq',
                    iqOrganization: 'faebc65955b24609b3c6bd053d07cf65',
                    iqStage: 'build',

                    iqScanPatterns: [
                       [scanPattern: 'target/*.jar'],
                       [scanPattern: 'target/dependency/*.jar']
                    ],

                    failBuildOnNetworkError: true,
                    failBuildOnScanningErrors: true,

                    reachability: [
                       failOnError: true,
                       logLevel: 'INFO',

                    javaAnalysis: [
                       enable: true,
                       entrypointStrategy: 'ACCESSIBLE_CONCRETE',

                       namespaces: [
                           [namespace: 'com.mycompany.app']
                       ],

                       includes: [
                           [pattern: 'target/my-app-*.jar'],
                           [pattern: 'target/dependency/*.jar']
                        ]
                   ]
                ]
           )
       }
    }


    }

    post {
        always {
            junit(
                allowEmptyResults: true,
                testResults: '**/target/surefire-reports/*.xml'
            )

            archiveArtifacts(
                artifacts: 'target/*.jar',
                allowEmptyArchive: true
            )
        }
    }
}

