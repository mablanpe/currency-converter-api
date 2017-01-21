#!groovy

node {
	// ------------------------------------
	// -- ETAPA: Compilar
	// ------------------------------------
	stage 'Compilar'
	
	// -- Configura variables
	echo 'Configurando variables'
	def mvnHome = tool 'Maven3'
	env.PATH = "${mvnHome}/bin:${env.PATH}"
	echo "var mvnHome='${mvnHome}'"
	echo "var env.PATH='${env.PATH}'"
	
	// -- Descarga código desde SCM
	echo 'Descargando código de SCM'
	sh 'rm -rf *'
	checkout scm
	
	// -- Compilando
	echo 'Compilando aplicación'
	sh 'mvn clean compile'
	
	// ------------------------------------
	// -- ETAPA: Test
	// ------------------------------------
	stage 'Test'
	echo 'Ejecutando tests'
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
	// -- ETAPA: Generando Javadocs
	// ------------------------------------
    checkout scm
    def mavenSettingsFile = " ${mvnHome}/conf/settings.xml"
	stage 'Build Web App'
	
	sh "mvn -s ${mavenSettingsFile} clean source:jar javadoc:javadoc checkstyle:checkstyle pmd:pmd findbugs:findbugs package"
	
	step([$class: 'ArtifactArchiver', artifacts: 'currency-converter-api/target/*.war'])
	step([$class: 'WarningsPublisher', consoleParsers: [[parserName: 'Maven']]])
	step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
	step([$class: 'JavadocArchiver', javadocDir: 'currency-converter-api/target/site/apidocs/'])
	
	// Use hudson.plugins.checkstyle.CheckStylePublisher if JSLint Publisher Plugin or JSHint Publisher Plugin is installed
	step([$class: 'hudson.plugins.checkstyle.CheckStylePublisher', pattern: '**/target/checkstyle-result.xml'])
	// In real life, PMD and Findbugs are unlikely to be used simultaneously
	step([$class: 'PmdPublisher', pattern: '**/target/pmd.xml'])
	step([$class: 'FindBugsPublisher', pattern: '**/findbugsXml.xml'])
	step([$class: 'AnalysisPublisher'])
	
	// ------------------------------------
	// -- ETAPA: Instalar
	// ------------------------------------
	stage 'Instalar'
	echo 'Instala el paquete generado en el repositorio maven'
	sh 'mvn install -Dmaven.test.skip=true'
	
	// ------------------------------------
	// -- ETAPA: Archivar
	// ------------------------------------
	stage 'Archivar'
	echo 'Archiva el paquete el paquete generado en Jenkins'
	step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar, **/target/*.war', fingerprint: true])
}
