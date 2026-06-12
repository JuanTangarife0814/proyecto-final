pipeline {
    agent any

    tools {
        maven 'Maven 3.9.16'
    }

    parameters {
        string(name: 'TOMCAT_WEBAPPS', defaultValue: '', description: 'Ruta completa de la carpeta webapps de Tomcat')
    }

    environment {
        APP_NAME = 'proyecto-final'
        WAR_FILE = 'target/proyecto-final.war'
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
                    bat 'mvn sonar:sonar -DskipTests -Dsonar.projectKey=%SONAR_PROJECT_KEY% -Dsonar.projectName=%SONAR_PROJECT_NAME%'
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        withSonarQubeEnv('SonarQube') {
                            def qualityGateStatus = bat(returnStdout: true, script: '''
@echo off
if not exist ".scannerwork\report-task.txt" (
    echo ERROR
    exit /b 0
)

for /f "tokens=2 delims==" %%A in ('findstr /B "ceTaskId=" .scannerwork\report-task.txt') do set TASK_ID=%%A

if not defined TASK_ID (
    echo ERROR
    exit /b 0
)

powershell -NoProfile -ExecutionPolicy Bypass -Command "$taskId = $env:TASK_ID; $serverUrl = $env:SONAR_HOST_URL.TrimEnd('/'); $token = $env:SONAR_TOKEN; $pair = $token + ':'; $bytes = [Text.Encoding]::ASCII.GetBytes($pair); $basic = [Convert]::ToBase64String($bytes); for ($i = 1; $i -le 30; $i++) { try { $task = Invoke-RestMethod -Headers @{ Authorization = ('Basic ' + $basic) } -Uri ($serverUrl + '/api/ce/task?id=' + $taskId); if ($task.task.status -eq 'SUCCESS') { $gate = Invoke-RestMethod -Headers @{ Authorization = ('Basic ' + $basic) } -Uri ($serverUrl + '/api/qualitygates/project_status?projectKey=%SONAR_PROJECT_KEY%'); Write-Output $gate.projectStatus.status; exit 0 } elseif ($task.task.status -in @('FAILED','CANCELED')) { Write-Output $task.task.status; exit 0 } } catch { } Start-Sleep -Seconds 5 } Write-Output 'TIMEOUT'; exit 0"
''').trim()

                            env.QUALITY_GATE_STATUS = qualityGateStatus
                            echo "SonarQube Quality Gate: ${env.QUALITY_GATE_STATUS}"

                            if (env.QUALITY_GATE_STATUS != 'OK') {
                                error("Quality Gate not OK: ${env.QUALITY_GATE_STATUS}")
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            when {
                expression { return params.TOMCAT_WEBAPPS != null && params.TOMCAT_WEBAPPS.trim() }
            }
            steps {
                bat 'copy /Y "%WAR_FILE%" "%TOMCAT_WEBAPPS%\\%APP_NAME%.war"'
            }
        }
    }

    post {
        success {
            echo 'Build OK. WAR generado y analizado correctamente.'
        }
        failure {
            emailext(
                to: "${NOTIFICATION_EMAIL}",
                subject: "Build Fallido - ${JOB_NAME} #${BUILD_NUMBER}",
                body: "El proyecto no paso el pipeline. Revisa Jenkins y SonarQube para ver el error.\n\nJob: ${JOB_NAME}\nBuild: ${BUILD_NUMBER}\nURL: ${BUILD_URL}"
            )
            echo 'Build fallido. Revisar compilacion, SonarQube o despliegue.'
        }
    }
}