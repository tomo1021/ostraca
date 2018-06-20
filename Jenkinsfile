def jenkins_path = "/var/lib/jenkins"
def tf_path      = "${jenkins_path}/terraform/build"
def ansible_path = "${jenkins_path}/ansible"
def terraform    =  "/usr/local/bin/terraform" 

def cgreen_name = ""

node {
    stage('scm'){
        checkout scm
    }

    stage("build & test"){
        sh "echo 'Building code and test.'"
    }

    stage('Confirm current green server'){
        //現在のGreenサーバの情報を取得
        dir("${tf_path}"){
            option = "\$3"
            // id = sh returnStdout: true, script: "${terraform} state show aws_lb_target_group_attachment.2anet_tgg_ata | grep target_id | awk '{print ${option}}'"
            id = sh returnStdout: true,\
                script: "${terraform} state show aws_lb_target_group_attachment.2anet_tgg_ata | grep target_id | awk '{print ${option}}'"
            sh "echo 'HELLO'"
            sh "echo ${id}"
            try{
                result = sh returnStdout: true,script: "${terraform} state show aws_instance.2anet_server1 | grep ${id}"
                cgreen_name="2anet_server1"
            }catch(exception){
                cgreen_name="2anet_server2"
            }
        }
        sh "echo ${cgreen_name}"
    }

    
    // stage('Destroy of the current green server'){
    //     //現行のGreenサーバを破棄
    //     dir("${tf_path}"){
    //         sh "${terraform} destroy -auto-approve -target=aws_instance.${cgreen_name} ./stage1"
    //     }
    // }
    stage('Destroy of the current green server'){
        //現在のgreenサーバを破棄
        dir("${tf_path}"){
            sh "${terraform} destroy -auto-approve -target=aws_instance.${cgreen_name} ./stage1"
        }
    }
    
    stage('Create new blue server instance'){
        //
        dir("${tf_path}"){
            sh "${terraform} apply -auto-approve ./stage1"
        }
    }

    stage('Probisioning for new blue server'){
        //Ansibleを使用して新しいblue serverを設定する
        dir("${tf_path}"){
            option = "\$3"
            ip = sh returnStdout: true,script:"${terraform} state show aws_instance.${cgreen_name} | grep '^public_ip' | awk '{print${option}}'| tr -d '\n'"
            
            sh "echo ${ip}"
        }
        dir("${ansible_path}"){
            sh "echo '[blue_server]' > ./hosts"
            sh "echo ${ip} >> ./hosts" 
            sh "ansible-playbook -i ./hosts --private-key=./2Anet.pem ./ostraca.yml"

        }
    }

    stage('Excute test for new blue server'){
        //
        sh "echo 'server test'"
    }

    stage('Switch the new blue server'){

    }
}