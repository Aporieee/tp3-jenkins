pipeline {
  agent any

  options {
    timeout(time: 10, unit: 'MINUTES')        // timeout global
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '5'))  // retention limitee : 5 builds
  }

  parameters {
    choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'prod'],
           description: 'Environnement cible (liste controlee)')
    string(name: 'VERSION', defaultValue: 'v1.0.0',
           description: 'Version (format vX.Y.Z)')
    booleanParam(name: 'DRY_RUN', defaultValue: true,
                 description: 'Simulation sans action reelle')
    string(name: 'CHANGE_REFERENCE', defaultValue: '',
           description: 'Reference de changement (ex: CHG-1234)')
  }

  stages {
    stage('Preparation') {
      steps {
        echo "Preparation - build #${env.BUILD_NUMBER}"
        sh 'mkdir -p artifacts'
      }
    }

    stage('Validation') {
      steps {
        script {
          if (!(params.ENVIRONMENT in ['dev','test','prod'])) {
            error("ENVIRONMENT invalide: '${params.ENVIRONMENT}'")
          }
          if (!(params.VERSION ==~ /^v\d+\.\d+\.\d+$/)) {
            error("VERSION invalide: '${params.VERSION}' (format vX.Y.Z)")
          }
          if (!(params.CHANGE_REFERENCE ==~ /^[A-Z]+-\d+$/)) {
            error("CHANGE_REFERENCE invalide: '${params.CHANGE_REFERENCE}' (format ABC-123)")
          }
          echo "Parametres valides."
        }
      }
    }

    stage('Execution') {
      steps {
        script {
          def sha = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
          currentBuild.displayName = "#${env.BUILD_NUMBER} - ${params.ENVIRONMENT} - ${sha}"
        }
        echo """Contexte :
  ENVIRONMENT      = ${params.ENVIRONMENT}
  VERSION          = ${params.VERSION}
  DRY_RUN          = ${params.DRY_RUN}
  CHANGE_REFERENCE = ${params.CHANGE_REFERENCE}"""
        sh '''
          printf 'build=%s\\ncommit=%s\\nenv=%s\\nversion=%s\\ndry_run=%s\\nchange=%s\\n' \
            "$BUILD_NUMBER" "$(git rev-parse --short HEAD)" \
            "$ENVIRONMENT" "$VERSION" "$DRY_RUN" "$CHANGE_REFERENCE" > artifacts/build-info.txt
          cat artifacts/build-info.txt
        '''
        archiveArtifacts artifacts: 'artifacts/build-info.txt', fingerprint: true
      }
    }
  }

  post {
    success { echo "Post-traitement : build ${currentBuild.displayName} OK" }
    failure { echo "Post-traitement : build en ECHEC (validation ou execution)" }
    always  { echo "Post-traitement : fin du pipeline (statut=${currentBuild.currentResult})" }
  }
}
