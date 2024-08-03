pipeline {
    agent any

    environment {
        //GCP_CREDENTIALS = credentials('gke-key-file')
        SE_SAND_ACC = credentials('SE_SAND_ACC')
        SE_SAND_SEC = credentials('SE_SAND_SEC')
        SE_SAND_PCC_URL = credentials('SE_SAND_PCC_URL')
        PCC_CLS_ADD = credentials('PCC_CLS_ADD')
    }

    stages {
        stage('Setup GCloud Auth') {
            steps {
                withCredentials([file(credentialsId: 'gke-key-file', variable: 'GCP_KEY_FILE')]) {
                    sh '''
                    gcloud auth activate-service-account --key-file="$GCP_KEY_FILE"
                    '''
                }
            }
        }

        stage('Get GKE Credentials') {
            steps {
                withCredentials([file(credentialsId: 'gke-key-file', variable: 'GCP_KEY_FILE')]) {
                    sh '''
                    gcloud container clusters get-credentials gke-sm --zone us-central1-c --project pcc-dev-sandbox
                    if ! kubectl get ns twistlock > /dev/null 2>&1; then
                      echo "Namespace 'twistlock' does not exist. Creating namespace 'twistlock'."
                      kubectl create ns twistlock
                    else
                      echo "Namespace 'twistlock' already exists."
                    fi
                    '''
                }
            }
        }

        stage('Deploy Defender') {
            steps {
                withCredentials([
                    string(credentialsId: 'SE_SAND_ACC', variable: 'SE_SAND_ACC'),
                    string(credentialsId: 'SE_SAND_SEC', variable: 'SE_SAND_SEC'),
                    string(credentialsId: 'SE_SAND_PCC_URL', variable: 'SE_SAND_PCC_URL'),
                    string(credentialsId: 'PCC_CLS_ADD', variable: 'PCC_CLS_ADD')
                ]) {
                    sh ''' 
                    curl -k -u "$SE_SAND_ACC:$SE_SAND_SEC" --output ./twistcli "$SE_SAND_PCC_URL/api/v1/util/twistcli"
                    chmod a+x ./twistcli
                    ./twistcli defender install kubernetes --address "$SE_SAND_PCC_URL" --cluster-address "$PCC_CLS_ADD" --cluster gke-sm --container-runtime containerd --namespace twistlock --output ./deployment.yaml --user "$SE_SAND_ACC" --password "$SE_SAND_SEC"
                    '''
                }
            }
        }
    }
}
