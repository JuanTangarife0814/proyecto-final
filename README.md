# Proyecto Final

Proyecto web Java basico empaquetado como WAR para desplegar en Tomcat.

## Estructura

- `pom.xml`: configuracion Maven.
- `src/main/java`: servlet de ejemplo.
- `src/main/webapp`: pagina inicial y configuracion web.
- `Jenkinsfile`: pipeline para Jenkins.

## Compilar

```bash
mvn clean package
```

El archivo final se genera en:

```bash
target/proyecto-final.war
```

## Jenkins

La carpeta `C:\Users\juand\OneDrive\Documentos\2026-1\Pruebas\Jenkins` solo contiene `jenkins.war`.
Eso sirve para ejecutar Jenkins, pero no reemplaza la configuracion del proyecto.

Para usar este examen necesitas dos cosas separadas:

1. Jenkins ejecutandose desde `jenkins.war`.
2. Este proyecto subido a GitHub con su `Jenkinsfile`.

### Como iniciar Jenkins con ese archivo

1. Asegurate de tener Java instalado.
2. Abre una terminal en la carpeta donde esta `jenkins.war`.
3. Ejecuta:

```bash
java -jar jenkins.war
```

4. Abre `http://localhost:8080`.
5. Instala los plugins de Git, Maven, SonarQube Scanner y Email Extension.

### Como usar el pipeline

1. Sube este proyecto a GitHub.
2. En Jenkins crea un job tipo Pipeline.
3. Configura el repo GitHub como fuente.
4. Jenkins tomara el `Jenkinsfile` de la raiz del proyecto.
5. El pipeline compila con Maven, analiza con SonarQube y puede copiar el WAR a Tomcat.
6. Por ahora no depende del webhook para no detenerse en `waitForQualityGate`.

### Variables que debes configurar en Jenkins

- `Maven 3.9.16`: nombre del Maven configurado en Jenkins.
- `SonarQube`: nombre de la conexion de SonarQube en Jenkins.
- `TOMCAT_WEBAPPS`: ruta de la carpeta `webapps` de Tomcat, por ejemplo `C:\apache-tomcat-9\webapps`.

## Probar en Tomcat

1. Copia `target/proyecto-final.war` a la carpeta `webapps` de Tomcat.
2. Inicia Tomcat.
3. Abre la aplicacion en el navegador.

## Siguiente paso para el examen

Con este proyecto base, luego se puede conectar Jenkins para:

1. Clonar desde GitHub.
2. Ejecutar `mvn clean package`.
3. Analizar con SonarQube.
4. Desplegar el WAR en Tomcat.
5. Enviar correo si falla el quality gate.

## Nota sobre el webhook

Si el webhook de SonarQube no funciona, el pipeline actual puede seguir avanzando sin bloquearse. Más adelante, cuando quieras dejar la integración completa, se puede volver a agregar la espera del Quality Gate y arreglar el webhook o usar una URL accesible desde SonarQube.