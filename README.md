# Infrastructure as Code with Security
Objective: To secure provision, configure, and manage cloud infrastructure using Iac and integrate security checks throughout the process

* Tools: Terraform(or AWS cloudformation), AWS config, checkov (for security scanning), and sentinel ( for policy as code).

 
 Step 1: I write Infrastructure Code Using Terraform (or AWS cloudFormation)
 
 1- I install Terraform:

 * I download and install Terraform on my local machine or CI/CD server.
 
 * I verify the installation:
 
 -> terraform -v
 
 2- I Set Up Provider configuration:
 
 * In my project directory, i create a main.tf file to configure my cloud provider (e.g, AWS).
 #
 * Example main.tf (for AWS).
 
 provider "aws" {
     region = "us-west-2"
 }
 
 3-  I define Resources:
 
 * I write code to define cloud resources, such as EC2 instances, VPCs, and S3 buckets.
 
 * Example for an EC2 instance:
 
 resource "aws_instance" "web_server" {
     ami           = "ami-12345678"
     instance_type = "t2.micro"
     tags = {
         name  = "webserver"
     }
    
 }
 
 
 4- I initialize and validate:
 
 * Initialize Terraform to dowload required providers:
 -> terraform init
 
 * i validate my configuration syntax:
 -> terraform validate
 
 5 - I plan and Apply
 
 * Generate an execution plan and review the proposed changes.
 -> terraform plan
 
 * Deploy the infrastructure.
 -> terraform apply
 
 
 Step 2: I integrate Security checks for configurations
 
 1- I set up AWS config (or equivalent) for security Compliance:
    
    * AWS COnfig provides security and compliance monitoring by assessing resources against best practices and standards.
    
    * I enable AWS Config in the AWS console or via CLI:
    -> aws configservice start-configuration-recorder --configuration-recorder-name default
    
    * I define rules in AWS Config to check configurations for compliance (e.g, ensuring S3 buckets are encrypted).
    
    
    2 - I set up Notifications for Non-compliant Resources:

    this is how to do it:
    step 1- Set up an SNS topic for notification
    1- Open the AWS Management console:
      -> Navigate to the Amozon SNS console
    2- Create a New SNS topic:
      -> click create topic
      -> choose the topic (e.g, configNonComplianceAlerts).
      -> add a description for better context.

    3- Configure topic settings:
      -> Set the default settings, or configure any specific policies if needed (e.g, restrict access only to AWS config).
      -> Click create topic.

    4- Add subscriptions to the topic:
      -> once the topic is created, click on the topic name to open details.
      -> Click create subscription.
      -> choose a protocol(e.g., Email or HTTPS) and enter the Endpoint (e.g., an email address for email notifications).
      -> Confirm the subscription if necessary (e.g., verify the email to enable notifications).

    step 3: Enable AWS config and create a config rule
    1- Go to AWS config console:
      -> Open AWS config console.
    2- Set up AWS Config(if not done already):
       -> In the config dashboard, click Get started if AWS Config is not set up in my account.
       -> Select the resources and regions i want to monitor. Choose the option to record all resources supported in this region or record specific resources as required.
       -> Ensure that i enable AWS Config rules and select an s3 bucket to store configuration history if prompted.

    3- Add a Compliance Rule:
       -> in the config console, go to rules and click add rule.
       -> browse or search for rule i will like to enforce (e.g., s3-bucket-public-read-prohibited to prevent public access on S3 buckets).
       -> Configure the rule settings if needed (e.g., specify the compliance checks).

    4- Configure the rule to use SNS for notifications:
      -> Under configure rule, select the SNS topic i created (configNonComplianceAlerts).
      -> Click save or Add rule.

    Step 3: Configure AWS config to trigger SNS notifications
    1- Enable SNS notifications for config changes:
      -> In the AWS config console, go to settings (found in the sidebar).
      -> Under General setting, locate amazon SNS topic settings.
      -> choose the SNS topic created in step 1
      (configNonComplianceAlerts) for notifications about compliance changes.
      -> save the settings.

  Step 4: Verify the configuration
  1- Test the rule:
      -> Trigger a change to test the rulle (e.g., make an s3 bucket public if using the s3-bucket-pucli-read-prohibited rule).
      -> AWS config will evaluate the rule periodically or on configuration change, depending on the rule type.

  2- Monitor for Notifications:
      -> When AWS config detects a non-compliant resource, it will send a message to the SNS topic, which will then forward it to the subscribed endpoint (e.g., an email or webhook).
      -> check my email(or other enpoinds) to ensure that the alert was received.

  step 5: Set up additional filtering
To filter specifi compliance changes or configure additional logic, i can create a lambda fucntion that triggers from SNS notifications. This lambda can then parse the message and take conditional actions based on the resource type or compliance state.

1- Create a Lambda function:
    -> go to the AWS lambda console
    -> click create function.
    -> Name the function (e.g.,configComplianceNotifier).
2- Add SNS trigger to lambda:
    -> in the lambda function configuration, add SNS as a trigger.
    -> select the configNonComplianceAlerts topic as the SNS source.
3- Write Lambda logic:
    -> Configure Lambda to parse the message from SNS and take specific actio, such as logging to Cloudwatch, sendind customized notifications, or calling APIs.
    
    * I use AWS SNS or Lambda to set up alerts when AWS Config detects a non-compliant resource.

    
    
    * For example, i create a Lambda function that triggers a notification to slack or email if an s3 is found to be publictly accessible.
    
    
    3 - I Automate Remediation of Compliance Issues
    
    * I configure AWS Config rules for auto-remediation. I use AWS Systems Manager to automatically resolve non-compliant resources based on predefined rules.
    
    
    
    Step 3: I Scan Iac Code for vulnerabilities using Checkov or Terraform Sentinel.
    
    1- I install Checkov:
    * I install checkov on my local machine or CI/CD pipeline:
    -> pip install checkov
    
    2- I run checkov scan:
    
    * I use CHeckov to scan Terraform files for security misconfigurations.
    
    * I run checkov on the IaC file in my project directory:
    -> checkov -d .
    
    * I review Results: Checkov generates a report higlighting vulnerabilities or configuration issues, such as public exposure of resources, unencrypted storage, or overly permissive IAM roles.
   
   3- I integrate Checkov with CI/CD pipeline:
   
    * I add a checkov scan stage to my CI/CD pipeline so each commit is scanned before merging.
     
    * Example in a jenkins pipeline (jenskinsfile)
    
    stage('checkov scan') {
       steps {
          sh 'checkov -d .'
      }
   }
   
   
   4- I enforce Policies with Terraform Sentinel ( if using Terraform Enterprise):
   
   * Sentinel enables me to define custom policies to restrict insecure configurations.
   
   * Example: A policy that restricts the deployment of public S3 buckets.
   
   * I add Sentinel policies to prevent deployment if policies fail.
   
   
   Step 4: I Deploy,Monitor,and Remediate Security Issues
   
   1- I deploy Infrastructure Using Terraform:
   
   *  I Apply the infrastructure configuration to deploy resources with security best practices in mind.
   
   * Example:
   -> terraform apply
   
   
   2- Continous monitoring:
   
   * I set UP CloudWatch or Prometheus: Monitor infrastructure performance and security.
   
   * I enable AWS GuardDuty ( or equivalent security service): GuardDuty conituously monitors for malicious activity in my environment.
   
   3 - I Remediate Detected Issues:
   
   * When vulnerabilities or misconfigurations are detected (via AWS Config, Checkov, or Sentinel), i remediate them immediatly:
   
      -> Manual Remediation: I adjust configuration in the Iac files, then i redeploy.
      
      -> Automated Remediation: I use Lambda functions or AWS systems Manager to automatically address common misconfigurations.
      
      
    4 - I implement Logging and Alerts:
    
    * I set up centralized logging (e.g, Aws CloudTrail, ELK stack) to store activity logs securely.
    
    * I configure alerts (Using SNS or Slack notifications) for critical security events, such as unauthorized access attempts.
    
    
    5 - I test Security Configurations:
    
    * I test security compliance by running Checkov scans and AWS Config evaluations reguarly
    
    
    * I update policies and security configurations based on findings to close any detected gaps.
    
    
    SummARY Checklist:
    
    1 - Terraform configuration: Written and applied securely
    2 - AWS Config: Enables with compliance rules and auto-remediation for critical configurations.
    3 - Checkov or sentinel: Configured in CI/CD to prevent insecure deployments.
    4- Monitoring and Logging: Configured for real-time security insights.
    5 - Alert and Remediation: Automated responses and notifications for high-severity issues.
    
    Note: By following this guide, i have securely implemented Iac setup with automated security check integrated at each stage, ensuring that my cloud infrastructure meets industry security standards in a real-world environment.
       
      
