pipeline {
    environment {
        AWSCRED="kaniko-aws-cred"
        AWS_REGION="us-east-2"
        ECR_REPO="237889007525.dkr.ecr.us-east-2.amazonaws.com"
        ECR_REPO_NAME="kaniko-demo-repo"
    }
    agent {
        kubernetes {
            label 'kaniko-demo'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: "jnlp"
    resources:
      requests:
        cpu: 300m
        memory: 400Mi
      limits:
        cpu: 400m
        memory: 800Mi
  - name: aws-cli
    image: amazon/aws-cli 
    resources:
      requests:
        cpu: 300m
        memory: 400Mi
      limits:
        cpu: 400m
        memory: 800Mi  
    command:
    - /bin/bash
    tty: true
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker/
      # when not using instance role
      - name: aws-secret
        mountPath: /root/.aws/
    env:
      - name: AWS_REGION
        value: us-west-2
    resources:
      requests:
        cpu: 500m
        memory: 400Mi
      limits:
        cpu: 500m
        memory: 800Mi       
    command:
    - /busybox/sh
    tty: true
  volumes:
    - name: docker-config
      configMap:
        name: docker-config
    # when not using instance role
    - name: aws-secret
      secret:
        secretName: aws-secret
'''
        }
    }
    stages {
        stage ('add aws credentials and build image') {
            steps {
                withCredentials([[
                $class: 'AmazonWebServicesCredentialsBinding',
                accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                credentialsId: 'kaniko-aws-cred',
                secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]){
                container('kaniko') {
                    sh '''
                    export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                    export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                    export AWS_DEFAULT_REGION=${AWS_REGION}
                    /kaniko/executor -f ${WORKSPACE}/kanikodemo/Dockerfile -c ${WORKSPACE} --force --destination=${ECR_REPO}/${ECR_REPO_NAME}:$BUILD_NUMBER --destination=${ECR_REPO}/${ECR_REPO_NAME}:latest
                    '''
                    }
                }
            }
        }
    }
}
