pipeline {
    agent any

    tools {
        maven 'Maven 3.9.16'
    }

    parameters {
        string(name: 'TOMCAT_WEBAPPS', defaultValue: 'C:\Users\juand\OneDrive\Documentos\2026-1\Pruebas\apache-tomcat-10.1.55-windows-x64\apache-tomcat-10.1.55\webapps', description: 'Ruta completa de la carpeta webapps de Tomcat')
    }

    environment {
        APP_NAME = 'proyecto-final'
        // WAR_FILE = 'target/proyecto-final.war'
        SONAR_PROJECT_KEY = 'proyecto-final'
        SONAR_PROJECT_NAME = 'proyecto-final'
        NOTIFICATION_EMAIL = 'juandiegotangarifemontoya@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat 'mvn sonar:sonar -DskipTests -Dsonar.projectKey=%SONAR_PROJECT_KEY% -Dsonar.projectName=%SONAR_PROJECT_NAME% -Dsonar.qualitygate.wait=true -Dsonar.qualitygate.timeout=300'
                }
            }
        }

        stage('Deploy to Tomcat') {
             when {
                 expression { return params.TOMCAT_WEBAPPS != null && params.TOMCAT_WEBAPPS.trim() }
             }
             steps {
                bat '''
                dir target
                for %%F in (target\\*.war) do copy /Y "%%F" "%TOMCAT_WEBAPPS%\\%APP_NAME%.war"
                '''
            }
        }

        // stage('Deploy to Tomcat') {
        //     when {
        //         expression { return params.TOMCAT_WEBAPPS != null && params.TOMCAT_WEBAPPS.trim() }
        //     }
        //     steps {
        //         bat 'copy /Y "%WAR_FILE%" "%TOMCAT_WEBAPPS%\\%APP_NAME%.war"'
        //     }
        // }
    }

    post {
        success {
            emailext(
                to: "${NOTIFICATION_EMAIL}",
                subject: "Build Exitoso - ${JOB_NAME} #${BUILD_NUMBER}",
                body: "El pipeline finalizo correctamente.\n\n" +
                    "Resultado: SUCCESS\n" +
                    "El proyecto compilo, paso la validacion de SonarQube y fue desplegado en Tomcat.\n\n" +
                    "Job: ${JOB_NAME}\n" +
                    "Build: ${BUILD_NUMBER}\n" +
                    "URL: ${BUILD_URL}\n" +
                    "SonarQube: http://localhost:9000/dashboard?id=${SONAR_PROJECT_KEY}\n" +
                    "Aplicacion: http://localhost:8081/${APP_NAME}"
            )
            echo 'Build OK. WAR generado, analizado y desplegado correctamente.'
        }

        failure {
            emailext(
                to: "${NOTIFICATION_EMAIL}",
                subject: "Build Fallido - ${JOB_NAME} #${BUILD_NUMBER}",
                body: "El pipeline fallo.\n\n" +
                    "Resultado: FAILURE\n" +
                    "El proyecto no paso alguna etapa: compilacion, SonarQube o despliegue.\n\n" +
                    "Job: ${JOB_NAME}\n" +
                    "Build: ${BUILD_NUMBER}\n" +
                    "URL: ${BUILD_URL}\n" +
                    "SonarQube: http://localhost:9000/dashboard?id=${SONAR_PROJECT_KEY}"
            )
            echo 'Build fallido. Revisar compilacion, SonarQube o despliegue.'
        }
    }

    // post {
    //     success {
    //         echo 'Build OK. WAR generado y analizado correctamente.'
    //     }
    //     failure {
    //         emailext(
    //             to: "${NOTIFICATION_EMAIL}",
    //             subject: "Build Fallido - ${JOB_NAME} #${BUILD_NUMBER}",
    //             body: "El proyecto no paso el pipeline. Revisa Jenkins y SonarQube para ver el error.\n\nJob: ${JOB_NAME}\nBuild: ${BUILD_NUMBER}\nURL: ${BUILD_URL}"
    //         )
    //         echo 'Build fallido. Revisar compilacion, SonarQube o despliegue.'
    //     }
    // }
}
