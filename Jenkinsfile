node {
    stage('Checkout') {
        checkout scm
    }

    stage('Staging') {
        sh '''
          docker rm -f samplerunning || true
          bash sample-app.sh
        '''
    }

    stage('Test staging') {
        sh '''
          sleep 5
          docker exec samplerunning python -c "import urllib.request; body = urllib.request.urlopen('http://localhost:5050/').read().decode(); assert 'You are calling me from' in body"
        '''
    }

    stage('Deploy production') {
        withCredentials([usernamePassword(credentialsId: 'webserver-login',
                                          usernameVariable: 'WEB_USER',
                                          passwordVariable: 'SSHPASS')]) {
            sh '''
              OPTS="-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null"
              HOST=$WEB_USER@192.0.2.10
              sshpass -e ssh $OPTS $HOST "rm -rf ~/sampleapp && mkdir -p ~/sampleapp"
              sshpass -e scp $OPTS -r sample-app.sh sample_app.py static templates $HOST:~/sampleapp/
              sshpass -e ssh $OPTS $HOST "docker rm -f samplerunning || true; cd ~/sampleapp && HOST_PORT=8080 bash sample-app.sh"
            '''
        }
    }
}