node {

    stage('Checkout') {
        checkout scm
    }
    
    stage('Preparation') {
        catchError(buildResult: 'SUCCESS') {
            sh 'docker stop samplerunning'
            sh 'docker rm samplerunning'
        }
    }
    stage('Build & deploy staging') {
        sh 'bash sample-app.sh'
    }
    stage('Results') {
        sh '''
        sleep 5
        curl http://172.17.0.4:5050/ | grep "You are calling me from 172.17.0.3"
        '''
    }

     stage('Deploy production') {
        sh '''
        export SSHPASS=vagrant
        OPTS="-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null"
        HOST=vagrant@192.0.2.10

        sshpass -e ssh $OPTS $HOST "rm -rf ~/sampleapp && mkdir -p ~/sampleapp"
        sshpass -e scp $OPTS -r sample-app.sh sample_app.py static templates $HOST:~/sampleapp/
        sshpass -e ssh $OPTS $HOST "docker rm -f samplerunning || true; cd ~/sampleapp && bash sample-app.sh"
        '''

        }
}