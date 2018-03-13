node{
			def DASID,DWSID,DDBSID,DDBS2ID,QVID,DVID,PVID,x,y,P1,P2,P3,P4,P5;
			def stack="${params.StackName}"
			def reg="${params.Region}"
			def qa="${params.QA_VPC_CIDR}"
			def dev="${params.Dev_VPC_CIDR}"
			def prod="${params.Prod_VPC_CIDR}"

      
                stage('Networking') 
        {       echo "create the whole networking stack and will output every resource created : vpc id’s , subnet id’s .... "
				withAWS(credentials:'CFN', region:"${reg}") { 
				echo "QA VPC CIDR: "+"${qa}" 
				echo "Dev VPC CIDR: "+"${dev}"
				echo "Prod VPC CIDR: "+"${prod}"
                
				def outputs = cfnUpdate(stack:stack +'-job1', params:["QAVNetworkCIDR=${qa}", "DEVVNetworkCIDR=${dev}", "PRODVNetworkCIDR=${prod}"], url:'https://s3-ap-northeast-1.amazonaws.com/aws-hcl-cfn-bucket/JOB1v5.template')
				outputs.each{ k, v -> println "${k}:${v}" }
             	DASID = outputs.find{ it.key == "DEVAppSubnetID" }.value
				DWSID = outputs.find{ it.key == "DEVWebSubnetID" }.value
				DDBSID = outputs.find{ it.key == "DEVDBSubnetID" }.value
				DDBS2ID = outputs.find{ it.key == "DEVDBSubnet2ID" }.value
				QVID = outputs.find{ it.key == "QAVPCID" }.value
				DVID = outputs.find{ it.key == "DEVVPCID" }.value
				PVID = outputs.find{ it.key == "PRODVPCID" }.value
				P1 = outputs.find{ it.key == "PRODPrivateRouteTableId" }.value
				P2 = outputs.find{ it.key == "DEVPrivateRouteTableId" }.value
				P3 = outputs.find{ it.key == "QAPrivateRouteTableId" }.value
				P4 = outputs.find{ it.key == "PRODPublicRouteTableId" }.value
				P5 = outputs.find{ it.key == "QAPublicRouteTableId" }.value
				P6 = outputs.find{ it.key == "DEVPublicRouteTableId" }.value
                   println "Dev app subnet id: " + DASID
                   println "Dev web subnet id: " + DWSID
				   println "Dev DB subnet id1: " + DDBSID
				   println "Dev DB subnet id2: " + DDBS2ID
				   println "====================================================================================================================="
				   println "QA VPC ID: " + QVID
				   println "Dev VPC ID: " + DVID
				   println "Prod VPC ID: " + PVID
				   
				   writeFile file: "/usr/lib/jenkins/networking.properties", text: "${DASID},${DWSID},${DDBSID},${DDBS2ID},${DVID}"
                    
               } 
        } 
		
		        stage('Security') 
        {        echo "create security"
					withAWS(credentials:'CFN', region:"${reg}") { 
                
				def outputs = cfnUpdate(stack:stack +'-job2', url:'https://s3-ap-northeast-1.amazonaws.com/aws-hcl-cfn-bucket/JOB2v1.template')
				outputs.each{ k, v -> println "${k}:${v}" }
         
                    
               } 
        } 
		
		        stage('Logging') 
        {        echo "create audit logging"
					withAWS(credentials:'CFN', region:"${reg}") { 
                
				def outputs = cfnUpdate(stack:stack +'-job3', url:'https://s3-ap-northeast-1.amazonaws.com/aws-hcl-cfn-bucket/JOB3v1-updated.template')
				outputs.each{ k, v -> println "${k}:${v}" }
         
                    
               }
               
               
               	 stage('VPC End Points') 
        {     
					echo "Running template 5, Creating VPC End Points"
					withAWS(credentials:'CFN', region:"${reg}") { 
			def outputs = cfnUpdate(stack:stack +'-job5', params:["VPCQA=${QVID}","VPCDEV=${DVID}","VPCPROD=${PVID}","PrivateRouteTablePROD=${P1}","PrivateRouteTableDEV=${P2}","PrivateRouteTableQA=${P3}",
				"PublicRouteTablePROD=${P4}","PublicRouteTableQA=${P5}","PublicRouteTableDEV=${P6}"	], url:'https://s3-ap-northeast-1.amazonaws.com/aws-hcl-cfn-bucket/JOB5.template')
				outputs.each{ k, v -> println "${k}:${v}" } 

					}
               }
               
               stage('IAM')
        {       
                	withAWS(credentials:'CFN', region:"${reg}") { 
                echo "Creating IAM Roles"
				def outputs = cfnUpdate(stack:stack +'-job6', url:'https://s3-ap-northeast-1.amazonaws.com/aws-hcl-cfn-bucket/JOB6.template')
				outputs.each{ k, v -> println "${k}:${v}" } 
        } 
        } 
		
        mail (to: "${params.Recipient}",
				subject: "AWS Cloud Formation Updates from Hipaa network deployment",
				body: 
                "HIPAA Network deployment is success. Please find the details below"+"\n"+"\n"+"QA VPC ID:          "+"${QVID}"+"\n"+"Dev VPC ID:         "+"${DVID}"+"\n"+
				"Prod VPC ID:        "+"${PVID}"+"\n"+"Dev App Subnet ID:        "+"${DASID}"+"\n"+"Dev Web Subnet ID:        "+"${DWSID}"+"\n"+"Dev DB Subnet ID:        "+"${DDBSID}"+
				"\n"+"Dev DB Subnet ID2:        "+"${DDBS2ID}")
        
                echo "Sending notification...."        
             
               
        
    }
}