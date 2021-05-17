node {
    files= ['deploy.yml']

  withCredentials([usernamePassword(credentialsId: 'prisma_cloud', passwordVariable: 'PC_PASS', usernameVariable: 'PC_USER')]) {
    PC_TOKEN = sh(script:"curl -s -k -H 'Content-Type: application/json' -H 'accept: application/json' --data '{\"username\":\"$PC_USER\", \"password\":\"$PC_PASS\"}' https://${AppStack}/login | jq --raw-output .token", returnStdout:true).trim()
      }
   stage('Clone repository') {
       checkout scm
    }

//  stage('Check image Git dependencies has no vulnerabilities') {
//        try {
//            withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
//                sh('chmod +x files/checkGit.sh && ./files/checkGit.sh')
//            }
//        } catch (err) {
//            echo err.getMessage()
//            echo "Error detected"
//			throw RuntimeException("Build failed for some specific reason!")
//        }
//    }

  stage('Check image Git dependencies wiht jenkins plugin') {
    try {
      echo 'before plugin  scanning'
     withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
     prismaCloudScanCode excludedPaths: '', explicitFiles: '', logLevel: 'debug', pythonVersion: '', repositoryName: 'evil.petclinic', repositoryPath: '.', resultsFile: 'prisma-cloud-scan-results.json'
      }         
     echo 'after  scanning'
       
     } finally {
       prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
        }
    
 }
	
	
    //$PC_USER,$PC_PASS,$PC_CONSOLE when Galileo is released. 
    stage('Apply Policy-as-Code for evilpetclinic') {
        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh('chmod +x files/addPolicies.sh && ./files/addPolicies.sh')
        }
    }

    //$PC_USER,$PC_PASS,$PC_CONSOLE when Galileo is released. 
    stage('Download latest twistcli') {
        withCredentials([usernamePassword(credentialsId: 'prisma_cloud', passwordVariable: 'PC_PASS', usernameVariable: 'PC_USER')]) {
            sh 'curl -k -u $PC_USER:$PC_PASS --output ./twistcli https://$PC_CONSOLE/api/v1/util/twistcli'
            sh 'sudo chmod a+x ./twistcli'
        }
    }

//    stage('Scan Image and Publish to Jenkins') {
//        try { // do something that fails
//            prismaCloudScanImage ca: '', cert: '', dockerAddress: 'unix:///var/run/docker.sock', ignoreImageBuildTime: true, image: 'rbenavente/evilpetclinic:latest', key: '', logLevel: 'debug', podmanPath: '', project: '', resultsFile: 'prisma-cloud-scan-results.json'
//           // currentBuild.result = 'SUCCESS'
//        } catch (Exception err) {
  //        currentBuild.result = 'UNSTABLE'
 //       } finally {
 //           prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
 //       }
 //       echo "RESULT: ${currentBuild.result}"
//    }

    stage('Scan image with twistcli and Publish to Jenkins') {
      try {
	    sh 'docker pull rbenavente/evilpetclinic:latest'  
            withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
           	prismaCloudScanImage ca: '', cert: '', dockerAddress: 'unix:///var/run/docker.sock', ignoreImageBuildTime: true, image: 'rbenavente/evilpetclinic:latest', key: '', logLevel: 'debug', podmanPath: '', project: '', resultsFile: 'prisma-cloud-scan-results.json'
		sh 'curl -k -u $TL_USER:$TL_PASS --output ./twistcli https://$TL_CONSOLE/api/v1/util/twistcli'
                sh 'sudo chmod a+x ./twistcli'
                sh "./twistcli images scan --u $TL_USER --p $TL_PASS --address https://$TL_CONSOLE --details rbenavente/evilpetclinic:latest"
            }
	 } finally {
            prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
        }
     //   catch (err) {
     //     echo err.getMessage()
     //       echo "Error detected"
//			throw RuntimeException("Build failed for some specific reason!")
  //      }
    }
  
stage('Scan K8s yaml manifest with Bridgecrew/checkov') {
  withDockerContainer(image: 'bridgecrew/jenkins_bridgecrew_runner:latest') {              
                  sh "/run.sh cadc031b-f0a7-5fe1-9085-e0801fc52131 https://github.com/rbenavente/shiftleftdemo"
               
            
        }
	}
stage("Scan Cloud Formation Template with API v2") {

        def response


        response = sh(script:"curl -sq -X POST -H 'x-redlock-auth: ${PC_TOKEN}' -H 'Content-Type: application/vnd.api+json' --url https://${AppStack}/iac/v2/scans --data-binary '@scan-asset.json'", returnStdout:true).trim()

        def SCAN_ASSET = readJSON text: response

        def SCAN_ID = SCAN_ASSET['data'].id
        def SCAN_URL = SCAN_ASSET['data']['links'].url

        //Upload files
        sh(script:"curl -sq -X PUT  --url '${SCAN_URL}' -T 'files/deploy.yml'", returnStdout:true).trim()

        //start the Scan
        response = sh(script:"curl -sq -X POST -H 'x-redlock-auth: ${PC_TOKEN}' -H 'Content-Type: application/vnd.api+json' --url https://${AppStack}/iac/v2/scans/${SCAN_ID} --data-binary '@scan-start-k8s.json'", returnStdout:true).trim()


        //Get the Status
        def SCAN_STATUS
        def STATUS

        //Need a Do-While loop here.   Haven't found a good syntax with Groovy in Jenkins
        response = sh(script:"curl -sq -H 'x-redlock-auth: ${PC_TOKEN}' -H 'Content-Type: application/vnd.api+json' --url https://${AppStack}/iac/v2/scans/${SCAN_ID}/status", returnStdout:true).trim()
        SCAN_STATUS = readJSON text: response
        STATUS = SCAN_STATUS['data']['attributes']['status']

        while  (STATUS == 'processsing'){
            response = sh(script:"curl -sq -H 'x-redlock-auth: ${PC_TOKEN}' -H 'Content-Type: application/vnd.api+json' --url https://${AppStack}/iac/v2/scans/${SCAN_ID}/status", returnStdout:true).trim()
            SCAN_STATUS = readJSON text: response
            STATUS = SCAN_STATUS['data']['attributes']['status']
            print "${STATUS}"

        }

        //Get the Results
        response = sh(script:"curl -sq -H 'x-redlock-auth: ${PC_TOKEN}' -H 'Content-Type: application/vnd.api+json' --url https://${AppStack}/iac/v2/scans/${SCAN_ID}/results", returnStdout:true).trim()
        def SCAN_RESULTS= readJSON text: response

        print "${SCAN_RESULTS}"

}
 
 
//    files.each { item ->
//        stage("Scan IaC file ${item} with twistcli") {
//            try {
//                withCredentials([usernamePassword(credentialsId: 'prisma_cloud', passwordVariable: 'PC_PASS', usernameVariable: 'PC_USER')]) {
//                    //sh "./twistcli iac scan --u $PC_USER --p $PC_PASS --asset-name "Jenkins IaC" --tags env:jenkins --compliance-threshold high --address https://$PC_CONSOLE --files files/${item}"
//                    sh "./twistcli iac scan --u $PC_USER --p $PC_PASS --type k8s --asset-name evilpetclinic --compliance-threshold medium --address https://$PC_CONSOLE files/${item}"
//                }
//            } catch (err) {
//                echo err.getMessage()
//                echo "Error detected"
//				throw RuntimeException("Build failed for some specific reason!")
//            }
//	    }
//    }

    stage('Deploy evilpetclinic') {
        sh 'kubectl create ns evil --dry-run -o yaml | kubectl apply -f -'
        sh 'kubectl delete --ignore-not-found=true -f files/deploy.yml -n evil'
        sh 'kubectl apply -f files/deploy.yml -n evil'
        sh 'sleep 30'
    }
    
   stage('Run bad Runtime attacks') {
        sh('chmod +x files/runtime_attacks.sh && ./files/runtime_attacks.sh')
    }
	
    stage('Run bad HTTP stuff for WAAS to catch') {
        sh('chmod +x files/waas_attacks.sh && ./files/waas_attacks.sh')
    }
}

