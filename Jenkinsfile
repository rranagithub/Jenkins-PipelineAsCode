node{
        
        if("${params.Environment}"=="Dev" && "${params.ApplicationTier}"=="Three Tier"){
        def x,y,z,RDS;
        def vpc=readFile '/usr/lib/jenkins/networking.properties'
        def values = vpc.split( ',' )
    
		def DASID=values[0]
		def DWSID=values[1]
		def DDBSID=values[2]
		def DDBS2ID=values[3]
		def DVID=values[4]
		
		def stack="${params.StackName}"
		def instance="${params.ApplicationName}"
		def IT="${params.ServerSize}"
		def reg="ap-northeast-1"

		        stage('Application Stack ') 
        {        echo "Create Application stack- ELB,EC2 and DB It will output the instance-id"
                    println "Dev app subnet id: " + DASID
                    println "Dev web subnet id: " + DWSID
				    println "Dev DB subnet id1: " + DDBSID
				    println "Dev DB subnet id2: " + DDBS2ID
				    println "Dev VPC ID: " + DVID
					withAWS(credentials:'CFN', region:"${reg}") { 
                
				def outputs = cfnUpdate(stack:stack +'-job4', params:["SubnetId=${DASID}","ELBSubnet=${DWSID}","DBSubnetId1=${DDBSID}","DBSubnetId2=${DDBS2ID}","devVPCId=${DVID}","InstanceType=${IT}","EC2NameTag=ansible-${instance}"], tags:["EC2Name=ansible-${instance}"], url:'https://s3-ap-northeast-1.amazonaws.com/aws-hcl-cfn-bucket/JOB4v7.template')
				outputs.each{ k, v -> println "${k}:${v}" } 
				x = outputs.find{ it.key == "AppServerPrivateIp" }.value
				y = outputs.find{ it.key == "AppServerPrivateDNS" }.value
                z = outputs.find{ it.key == "ELBDNSName" }.value
                RDS = outputs.find{ it.key == "RDSEndpoint" }.value
					}
               }
               
               stage('Ansible Tower')
        {       echo "Calling Ansible Job"
                build job: 'Environment-Provisioning-Pipeline', 
				parameters: [
                        [$class:'StringParameterValue', name:'cfn_ip_address', value:x],
                        [$class:'StringParameterValue', name:'privateDNS', value:y],
                        [$class:'StringParameterValue', name:'RDS End Point', value:RDS]
                        ]
        } 
        
                stage('Notification'){
                    echo "Sending notification...." 
                    mail (to: "${params.Recipient}",
				subject: "AWS Cloud Formation Updates from Application Deployment",
				body: "EC2 Instance and RDS has been created sucessfully..."+"\n"+"\n"+"EC2 Instance Private IP:       ${x}"+"\n"+"EC2 Instance Private DNS:       ${y}" +"\n"+"ELB URL:      "+"${z}/${instance}/"+"\n"+"RDS End Point:      "+"${RDS}")
        
                }
                     
        }    else{
            echo "Check you have selected right environment and right application tier"
        }
        
}
