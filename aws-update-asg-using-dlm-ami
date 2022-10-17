"""

This Lambda function was created to automate updating an AWS ASG & Launch Configuration for the 
for a specific EC2 instance with the latest AMI created by an AWS DLM backup policy.

"""
import boto3
import botocore
import logging
import json
import datetime

# Setup logging
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# Serves as the function's main method within the Lambda environment
def lambda_handler(event, context):

    #  Declare global vars
    targetAmi = '<ec2-instance-naming-convention>'
    amiOwner = ['<ami-ownerID>']
    launchConfiguration = ['<ec2-instance-naming-convention><launch-config-name>']
    # Launch Config vars
    launchConfigurationName = '<ec2-instance-naming-convention>-launch-config-lambda-' + datetime.datetime.today().strftime('%Y-%m-%d')
    targetEc2AsgName = '<ec2-instance-naming-convention>-asg'

    print("check 1 - lambda started")
    images = get_targetEc2_ami(targetEc2Ami, amiOwner)
    print(images)
    targetEc2Asg = get_launch_configuration(launchConfiguration)
    print(targetEc2Asg)
    isLatestAMI = check_latest_ami(images, targetEc2Asg)
    if (isLatestAMI == False):
        print('ASG will be updated with the latest AMI created by the Ec2 instance DLM policy')
        createLC = create_launch_config(images, targetEc2Asg, launchConfigurationName)
        if (createLC == True):
            updateAsg = update_asg_config(launchConfigurationName, targetEc2AsgName)
            if (updateAsg == True):
                deleteOldLc = delete_old_launch_config
            elif (update_asg_config == False):
                print('updating ASG returned False')
            else:
                print("unknown error updating ASG")
        elif (createLC == False):
            print('create_launch_config returned false')
        else:
            print('error creating launch config')
        delete_old_launch_config(targetEc2Asg)
    elif (isLatestAMI == True):
        print('error or latest AMI already used in instance's autoscaling group')
    else:
        print('Unexpected script behavior, isLatestAMI = check_latest_ami did not return true or false')
    return {
        'statusCode': 200,
        'body': json.dumps(images)
    }

# Reteirves the latest targetEc2 AMI created by our Data LifeCycle Manager (DLM) policy
def get_targetEc2_ami(targetEc2Ami, amiOwner):
    ec2 = boto3.client('ec2')
    try:
        response = ec2.describe_images(
            Owners=amiOwner,
            Filters=[ 
                {
                    'Name': 'tag:Name', 
                    'Values': [targetEc2Ami]
                },
            ],
        )
        logger.info('get_targetEc2_ami boto3 response: ' + targetEc2Ami)
    except:
        logger.error('get_targetEc2_ami ec2.describe images failed')
    mostRecentDate =  datetime.datetime.today() - datetime.timedelta(days=7)
    mostRecentAmi = ''
    # Iterate through each Image returned and look for mopst recent AMI
    for image in response['Images']:
        currentImageDateStr = image['CreationDate']
        print(currentImageDateStr)
        currentImageDateFrmt = datetime.datetime.strptime(currentImageDateStr, "%Y-%m-%dT%H:%M:%S.%fZ")
        if (currentImageDateFrmt > mostRecentDate):
            mostRecentAmi = image['ImageId']
            #logger.info(mostRecentAmi)
        else:
            logger.info('issue with finding latest AMI')
    logger.info('get_targetEc2_ami returned: ' + mostRecentAmi)
    return mostRecentAmi

# Retrieves the launch configuration attached to the targetEc2 Auto Scaling Group (ASG)
def get_launch_configuration(launchConfiguration):
    try:
        asg = boto3.client('autoscaling')
        returnedAsgs = asg.describe_launch_configurations(
            LaunchConfigurationNames=launchConfiguration
        )
        print(returnedAsgs)
        return returnedAsgs
    except:
        print(returnedAsgs)
        logger.error('error findign old targetEc2 launch configuration')

# Compares version of latest targetEc2 backup AMI to version in the targetEc2 ASG and returns true or false
def check_latest_ami(images, targetEc2Asg):
    latestAMI = images
    for asg in targetEc2Asg['LaunchConfigurations']:
        oldAmi = asg['ImageId']
    print('AMIs from check, Latest:' + latestAMI + ' Old AMI: ' + oldAmi)
    if latestAMI != oldAmi:
        print('new AMI found, set latestAMI = false')
        return False
    elif latestAMI == oldAmi:
        print('latest AMI is equal to current AMI in launch config')
        return True
    else:
        print('something went wrong when checking if latest AMI equals the launch config')

# Creates a new launch configuration using the old launch config info updated with the new targetEc2 AMI ID
def create_launch_config(images, targetEc2Asg, launchConfigurationName):
    lcfg = boto3.client('autoscaling')
    try:
        response = lcfg.create_launch_configuration(
            LaunchConfigurationName=launchConfigurationName,
            ImageId=images,
            KeyName=targetEc2Asg['LaunchConfigurations'][0]['KeyName'],
            SecurityGroups=targetEc2Asg['LaunchConfigurations'][0]['SecurityGroups'],
            UserData=targetEc2Asg['LaunchConfigurations'][0]['UserData'],
            InstanceType=targetEc2Asg['LaunchConfigurations'][0]['InstanceType'],
            BlockDeviceMappings=targetEc2Asg['LaunchConfigurations'][0]['BlockDeviceMappings'],
            InstanceMonitoring={'Enabled': False},
            IamInstanceProfile=targetEc2Asg['LaunchConfigurations'][0]['IamInstanceProfile'],
            AssociatePublicIpAddress=False,
        )
        print('launch configuration created')
        return True
    except:
        print('create_launch_config error')
        return False
# Updates the existing targetEc2 ASG with the newly created launch config
def update_asg_config(launchConfigurationName, targetEc2AsgName):
    updateAsg = boto3.client('autoscaling')
    try:
        response = updateAsg.update_auto_scaling_group(
            AutoScalingGroupName=targetEc2AsgName,
            LaunchConfigurationName=launchConfigurationName
        )
        print(response)
        return True
    except:
        print('updating ASG with new launch configuration failed')
        return False

# Deletes old launch configuration after the targetEc2 ASG is updated with the new one
def delete_old_launch_config(targetEc2Asg):
    #try:
    #    deleteAsg = boto3.client.delete_launch_configuration(
    #        LaunchConfigurationName=targetEc2Asg['LaunchConfigurations'][0]['LaunchConfigurationName'],
    #    )
    #except:
    #    logger.error('error deleting old launch config')
    print('placeholder for because deleting things is scary')
