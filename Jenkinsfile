pipeline {
    agent any

    environment {
        PROJECT_ID = 'devioz-pe-dev-analitica'
        NAME_SECRET = 'aws_Cred'
        GCP_SERVICE_ACCOUNT = 'devioz-corporativo-gcp-devops-analitica-dev'
        GCP_LOCATION = 'us-central1'
        NAME_BUCKET_GCP = 'mi-bucket-gcp-2'
        NAME_BUCKET_S3 = 'mi-bucket-aws-1'
        NAME_TRANSFER = 'PRUEBA1'
        NAME_BUCKET_AWS = 'mi-bucket-aws-1'
    }

    stages {
        stage('Descarga de Fuentes') {
            steps {
                script {
                    deleteDir()
                    checkout scm
                }
            }
        }

        stage('Activando Service Account') {
            steps {
                withCredentials([file(credentialsId: "${GCP_SERVICE_ACCOUNT}", variable: 'SECRET_FILE')]) {
                    sh """\$(gcloud auth activate-service-account --key-file=\$SECRET_FILE)"""
                }
            }
        }
        
        stage('Create Bucket in GCP') {
            steps {
                script {
                    sh "gcloud config set project ${PROJECT_ID}"
                    def bucketExists = sh(script: "gsutil ls gs://${NAME_BUCKET_GCP}", returnStatus: true)
                    if (bucketExists != 0) {
                        sh "gsutil mb -p ${PROJECT_ID} -l ${GCP_LOCATION} gs://${NAME_BUCKET_GCP}"
                    } else {
                        echo "El bucket gs://${NAME_BUCKET_GCP} ya existe. No se creará otro."
                    }
                }
            }
        }
        
        stage('Transferencia de datos de AWS a GCP') {
            steps {
                script {
                    // Descargar los datos de AWS S3
                    sh "aws s3 cp s3://${NAME_BUCKET_AWS} /tmp/${NAME_BUCKET_AWS} --recursive"
                    
                    // Subir los datos a Google Cloud Storage
                    sh "gsutil -m cp -r /tmp/${NAME_BUCKET_AWS} gs://${NAME_BUCKET_GCP}"
                }
            }
        }

        stage('Limpiando Workspace') {
            steps {
                deleteDir()
            }
        }
    }
}


