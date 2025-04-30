pipeline {
    agent any

    environment {
        GIT_CREDENTIALS_ID = 'github-creds' 
        GITHUB_REPO = 'dqvoleto11/testForJenkins'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    BRANCH_NAME_CLEAN = env.GIT_BRANCH.replaceAll('origin/', '')
                    TAG_NAME = "${BRANCH_NAME_CLEAN}-${new Date().format('yyyy-MM-dd')}"
                }
            }
        }

        stage('Instalar dependencias') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build del proyecto') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Empaquetar artefacto') {
            steps {
                sh 'zip -r build.zip .'
                archiveArtifacts artifacts: 'build.zip', fingerprint: true
            }
        }

        stage('Crear tag en GitHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${env.GIT_CREDENTIALS_ID}",
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_PASSWORD'
                    )
                ]) {
                    sh """
                        git config user.name "jenkins"
                        git config user.email "jenkins@example.com"
                        git tag -a ${TAG_NAME} -m "Build generado por Jenkins"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GITHUB_REPO}.git ${TAG_NAME}
                    """
                }
            }
        }

        stage('Subir a GitHub Release') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${env.GIT_CREDENTIALS_ID}",
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_TOKEN'
                    )
                ]) {
                    script {
                        sh """
                            curl -X POST https://api.github.com/repos/${GITHUB_REPO}/releases \\
                              -H "Authorization: token ${GIT_TOKEN}" \\
                              -H "Content-Type: application/json" \\
                              -d '{
                                "tag_name": "${TAG_NAME}",
                                "name": "${TAG_NAME}",
                                "body": "Build automático desde Jenkins",
                                "draft": false,
                                "prerelease": false
                              }'
                        """
                        sh """
                            UPLOAD_URL=$(curl -s -H "Authorization: token ${GIT_TOKEN}" https://api.github.com/repos/${GITHUB_REPO}/releases/tags/${TAG_NAME} | jq -r '.upload_url' | sed 's/{?name,label}//')
                            curl -X POST "$UPLOAD_URL?name=build-${TAG_NAME}.zip" \\
                              -H "Authorization: token ${GIT_TOKEN}" \\
                              -H "Content-Type: application/zip" \\
                              --data-binary @build.zip
                        """
                    }
                }
            }
        }
    }
}
