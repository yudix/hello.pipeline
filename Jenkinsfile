

node {
    environment {
        device_ver =  "${env.device_ver}"
        envs =        "${env.envs}"
        signed_from = "${env.signed_from}"
        USE_DM_VERITY = true
        displayName = readFile('/mnt/data/smbshare/share/build_id/build-id-jenkins-nougat.txt')
    }
   echo device_ver

/**    
    tools {
        jdk 'JDK1.8'
    }
*/

   // agent any
    // agent {
    //     label 'AOSP'
    // }
    

        
        stage('get gerrit repo manifest') {
                //git 'http://gerrit2@gerrit02.kaymera.com:8080/Nougat/platform/manifest'
                sh '''
                    cd /mnt/data/projects/Kaymera_Roms/Nougat
                    #sudo repo sync
                    '''
            
        }
        
        stage ('build'){
                sh 'sudo /mnt/data/BuildScripts/copyKeys.sh Nougat'
                sh 'sudo /mnt/data/SharedScripts/copyAndroidAppsFromSMB.sh Nougat '+ displayName
                currentBuild.displayName = currentBuild.displayName + "-" + device_ver
            
        }
        
        stage ('Run AOSP_build_Nougat') {
            def deviceVerList = Arrays.asList(device_ver.split("\\s*,\\s*"));
            def project_name = "AOSP_build_Nougat";
            def jobs_to_build = [:]

            for (deviceVer in deviceVerList) {
                jobs_to_build[deviceVer] = {
                    build   job: project_name,
                            parameters: 
                                [
                                    [$class: 'StringParameterValue', 
                                        name: 'BN', 
                                        value: displayName],
                                    [$class: 'StringParameterValue', 
                                        name: 'device_ver', 
                                        value: deviceVer]
                                ]
                }
            }
            parallel jobs_to_build
        }
       
        stage ('undo Keys'){
                sh 'sudo /mnt/data/BuildScripts/undoKeys.sh Nougat'
        }
        
        stage ('email notification') {
                build   job: 'RomSigning-Nougat', 
                        parameters: [
                            string(name: 'device_ver',  value: device_ver), 
                            string(name: 'envs',        value: envs),  
                            string(name: 'signed_from', value: signed_from) ],
                        propagate: true
        }
        

    
    post {
        failure {
            mail to: 'yehudag@kaymera.com',
                 subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Something is wrong with ${env.BUILD_URL}"
        }
    }
    
}

