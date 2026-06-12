pipeline {
    agent any

    tools {
        maven 'Maven 3.9.16'
    }

    environment {
        APP_NAME = 'proyecto-final'
        WAR_FILE = 'target/proyecto-final.war'
        SONAR_PROJECT_KEY = 'proyecto-final'
        SONAR_PROJECT_NAME = 'proyecto-final'
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
                    bat 'mvn sonar:sonar -DskipTests -Dsonar.projectKey=%SONAR_PROJECT_KEY% -Dsonar.projectName=%SONAR_PROJECT_NAME%'
                }
            }
        }

        stage('Deploy to Tomcat') {
            when {
                expression { return env.TOMCAT_WEBAPPS != null && env.TOMCAT_WEBAPPS.trim() }
            }
            steps {
                bat 'copy /Y %WAR_FILE% "%TOMCAT_WEBAPPS%\\%APP_NAME%.war"'
            }
        }
    }

    post {
        success {
            echo 'Build OK. WAR generado y analizado correctamente.'
        }
        failure {
            echo 'Build fallido. Revisar compilacion, SonarQube o despliegue.'
        }
    }
}