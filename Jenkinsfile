#!groovy

node {
   // ------------------------------------
   // -- ETAPA: Compilar
   // ------------------------------------
   stage 'Compilar'
   
   // -- Configura variables maven
   echo 'Configurando variables'
   def mvnHome = tool 'M3'
   env.PATH = "${mvnHome}/bin:${env.PATH}"
   echo "var mvnHome='${mvnHome}'"
   echo "var env.PATH='${env.PATH}'"
   
   // -- Descarga código desde SCM (Source Control Management)
   echo 'Descargando código de SCM'
   checkout scm
   
   // -- Compilando
   echo 'Compilando aplicación'
   sh 'mvn clean compile'
   
   // ------------------------------------
   // -- ETAPA: Pruebas
   // ------------------------------------
   stage 'Pruebas'
   echo 'Ejecutando pruebas'
   try{
      sh 'mvn verify'
      step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
   }catch(err) {
      step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
      if (currentBuild.result == 'UNSTABLE')
         currentBuild.result = 'FAILURE'
      throw err
   }
   
   // ------------------------------------
   // -- ETAPA: Instalar desarrollo
   // ------------------------------------
   stage 'Instalar repo desa'
   echo 'Instala el paquete generado en el repositorio maven'
   sh 'mvn install -Dmaven.test.skip=true'
   
   // ------------------------------------
   // -- ETAPA: Archivar
   // ------------------------------------
   stage 'Archivar'
   echo 'Archiva el paquete el paquete generado en Jenkins'
   step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar, **/target/*.war', fingerprint: true])    

    // ------------------------------------
    // -- ETAPA: Instalar en release
    // ------------------------------------

    // we want to pick up the version from the pom
    def pom = readMavenPom file: 'pom.xml'
    def version = pom.version.replace("-SNAPSHOT", ".${currentBuild.number}")


    // Mark the code build 'stage'....
    stage 'Build'
    // Run the maven build this is a release that keeps the development version 
    // unchanged and uses Jenkins to provide the version number uniqueness
    sh "${mvnHome}/bin/mvn -DreleaseVersion=${version} -DdevelopmentVersion=${pom.version} -DpushChanges=false -DlocalCheckout=true -DpreparationGoals=initialize release:prepare release:perform -B"
    // Now we have a step to decide if we should publish to production 
    // (we just use a simple publish step here)
    input 'Publish?'
    stage 'Publish'
    // push the tags (alternatively we could have pushed them to a separate
    // git repo that we then pull from and repush... the latter can be 
    // helpful in the case where you run the publish on a different node
    sh "git push ${pom.artifactId}-${version}"
    // we should also release the staging repo, if we had stashed the 
    //details of the staging repository identifier it would be easy




}