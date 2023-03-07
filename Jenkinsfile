def PREFIX = 'https://mst-support.sw.siemens.com/?RelayState=/dvt'
def MST = "/home/habdelgaw"
def USERNAME = "svc_icvs_dvt"
def SERVER = "mst-support.sw.siemens.com"
def mailRecipients = '''yara.amrallah@siemens.com, \
                    hanaa.abdelgawad@siemens.com, \
                    '''
def success_function(log, name, cloud_loc, filename){
    sh "echo PASS >> ${log} 2>&1"
    sh 'sleep 120'
    sh "rm -f ${MST}/queue/${name}/${cloud_loc}/${filename}"
    sh "find ${MST}/queue/${name}/ -empty -type d -delete"
    sh "echo '${PREFIX}/${cloud_loc}/${filename} >>/tmp/url.log'"
    def html_content = "${PREFIX}/${cloud_loc}/${filename}"
    emailext mimeType: 'text/html',
    body: """
    \${SCRIPT, template="groovy-html.template"}
    ${html_content}
    """,
    subject: "[MST] Upload of ${cloud_loc}/${filename} PASS",
    to: "${mailRecipients}",
    replyTo: "${mailRecipients}",
    recipientProviders: [
        [$class: 'CulpritsRecipientProvider']
    ]
}

def fail_function(log, name, cloud_loc, filename){
    sh "echo FAIL >> ${log} 2>&1"
    
    sh "mv ${MST}/queue/${name} ${MST}/fail/"
    
    emailext mimeType: 'text/html',
    subject: "[MST] Upload of ${cloud_loc}/${filename} FAIL",
    to: "${mailRecipients}",
    replyTo: "${mailRecipients}",
    recipientProviders: [
        [$class: 'CulpritsRecipientProvider']
    ]
}

pipeline {
    agent any
    options {
        timeout(time: 60, unit: 'MINUTES')
    }
    stages {
        stage("MST") {
            steps {
              script {
                    sshagent(['mst-keys']) {
                        try {
                            sh 'sleep 60'
                            sh "cd ${mst}/queue"
                            available_names = sh(returnStdout: true, script: "ls").trim().split()
                        
                            for (name in available_names){
                                log = "${MST}/logs/${}.log"
                                is_directory = sh(script: "test -d ${name}", returnStdout: true).trim()
                                directory_files = sh(script: "ls -A ${name}", returnStdout: true).trim().split()
                                if( !is_directory || directory_files.length == 0) {//not directory or empty directory
                                    sh(script: "echo '${name} is not a directory or empty' >> ${log} 2>&1").trim()
                                    continue
                                }
                                mailRecipients = mailRecipients + name + "@wv.mentorg.com, \\"
                                folders_list=sh(returnStdout: true, script: "find ${name} -type d -links 2").trim().split() 
                                        
                                for (folder in folders_list ){
                                    sh "echo ${folder}"
                                    cloud_loc=sh(returnStdout: true, script: "echo ${folder} |awk -F'${name}' '{print \$2}'").trim()
                                    sh "echo ${cloud_loc}"
                                    filename_list=sh(returnStdout: true, script: "ls ${folder} -type d -links 2").trim().split() 
                                    sh "sftp ${USERNAME}@${SERVER} >> ${log} 2>&1 <<!EOF!"
                                    sh "mkdir ${cloud_loc}"
                                    sh "cd ${cloud_loc}"
                                    for (filename in filename_list){
                                        sh "echo ${name} ${cloud_loc} ${filename} > ${log} 2>&1"
                                        sh "echo 'Processing ${name}/${cloud_loc}/${filename} -> ${cloud_loc}'"
                                        sh "put ${filename}"
                                        check_flag=sh(returnStdout: true, script: "test \$1").trim()
                                        if (check_flag == 0)
                                            success_function(log, name, cloud_loc, filename)
                                        else
                                            fail_function(log, name, cloud_loc, filename)
                                    }
                                    sh "exit"
                                }
                            }//end names
                        } catch (err) {
                            echo "Catching[2]: ${err}"
                            currentBuild.result = 'FAILURE'
                            throw err
                        }
                    }
                }
            }
        }
    }
}
