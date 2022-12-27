pipeline {
  agent any
  parameters {
    choice (name: 'chooseNode', choices: ['Green', 'Blue'], description: 'Choose which Environment to Deploy: ')
  }
  environment {
    listenerARN = 'arn:aws:elasticloadbalancing:us-east-1:007591908515:listener/app/aloadbalancer/ff2b0b45934a7313/3a52d6c18097292c'
    blueARN = 'arn:aws:elasticloadbalancing:us-east-1:007591908515:targetgroup/bluetarget/eaea52c5837220aa'
    greenARN = 'arn:aws:elasticloadbalancing:us-east-1:007591908515:targetgroup/green/4b186d98078d0164'
  }
  stages {
    stage('Deployment Started') {
      parallel {
        stage('Green') {
          when {
            expression {
              params.chooseNode == 'Green'
            }
          }
          stages {
            stage('Offloading Green') {
              steps {
                sh """aws elbv2 modify-listener --listener-arn ${listenerARN} --default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 0 },{"TargetGroupArn": "${blueARN}", "Weight": 1 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'"""
              }
            }
            stage('Deploying to Green') {
              steps {
                sh '''scp -r index.html ec2-user@54.226.195.153:/usr/share/nginx/html/
                ssh -t ec2-user@54.226.195.153 -p 22 << EOF 
                sudo service nginx restart
                '''
              }
            }
            stage('Validate and Add Green for testing') {
              steps {
                sh """
                if [ "\$(curl -o /dev/null --silent --head --write-out '%{http_code}' http://54.226.195.153/)" -eq 200 ]
                then
                    echo "** BUILD IS SUCCESSFUL **"
                    curl -I http://54.226.195.153/
                    aws elbv2 modify-listener --listener-arn ${listenerARN} --default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 0 },{"TargetGroupArn": "${blueARN}", "Weight": 1 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'
                else
	                echo "** BUILD IS FAILED ** Health check returned non 200 status code"
                    curl -I http://54.226.195.153/
                exit 2
                fi
                """
              }
            }
          }
        }
        stage('Blue') {
          when {
            expression {
              params.chooseNode == 'Blue'
            }
          }
          stages {
            stage('Offloading Blue') {
              steps {
                sh """aws elbv2 modify-listener --listener-arn ${listenerARN} --default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 1 },{"TargetGroupArn": "${blueARN}", "Weight": 0 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'"""
              }
            }
            stage('Deploying to Blue') {
              steps {
                sh '''scp -r index.html ec2-user@52.201.228.94:/usr/share/nginx/html/
                ssh -t ec2-user@52.201.228.94 -p 22 << EOF 
                sudo service nginx restart
                '''
              }
            }
            stage('Validate Blue and added to TG') {
              steps {
                sh """
                if [ "\$(curl -o /dev/null --silent --head --write-out '%{http_code}' http://52.201.228.94/)" -eq 200 ]
                then
                    echo "** BUILD IS SUCCESSFUL **"
                    curl -I http://52.201.228.94/
                    aws elbv2 modify-listener --listener-arn ${listenerARN} --default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 1 },{"TargetGroupArn": "${blueARN}", "Weight": 1 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'
                else
	                echo "** BUILD IS FAILED ** Health check returned non 200 status code"
                    curl -I http://52.201.228.94/
                exit 2
                fi
                """
              }
            }
          }
        }
      }
    }
  }
}
