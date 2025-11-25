pipeline {
    agent any

    environment {
        PROJECT_NAME = "pipeline-test"
        SONARQUBE_URL = "http://54.85.101.56:9000"
        SONARQUBE_TOKEN ="sqa_b6b88e9d74b195f593d37a178668d7e5cc78c86c"
        TARGET_URL = "http://172.31.19.120:5000" //  ←✔ ip del servidor cuando lo instalas
    }

    stages {

        stage('Install Python') {
            steps {
                sh '''
                    apt update
                    apt install -y python3 python3-venv python3-pip
                '''
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install Flask==2.2.5 Werkzeug==2.2.3 Jinja2==3.1.2 itsdangerous==2.1.2 click==8.1.3
                '''
            }
        }

        stage('Python Security Audit') {
            steps {
                sh '''
                    . venv/bin/activate
                    pip install pip-audit
                    mkdir -p dependency-check-report
                '''

                script {
                    def exitCode = sh(
                        script: '. venv/bin/activate && pip-audit -f markdown -o dependency-check-report/pip-audit.md',
                        returnStatus: true
                    )

                    if (exitCode != 0) {
                        echo "pip-audit encontró vulnerabilidades, pero se continúa."
                    } else {
                        echo "pip-audit no encontró vulnerabilidades."
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SONAR'  //  ←✔ Nombre de la instalacion
                    withSonarQubeEnv('SonarServer') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=${PROJECT_NAME} \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=${SONARQUBE_URL}
                        """
                    }
                }
            }
        }

        stage('Dependency Check') {
            steps {//  ←✔ Copia y pega la key en api key
                dependencyCheck(
                    additionalArguments: "--scan . --format HTML --out dependency-check-report --enableExperimental --enableRetired --nvdApiKey fbe5db98-60ff-4305-be32-13e687cf7bbc", 
                    odcInstallation: 'DependencyCheck'  //  ←✔ Nombre de la instalacion
                )
            }
        }
    }  //  ←✔ Cierre correcto de `stages`

    post {
        always {
            archiveArtifacts artifacts: 'dependency-check-report/**', fingerprint: true
        }
        success {
            echo "Build success"
        }
        failure {
            echo "Build failed"
        }
    }

}
