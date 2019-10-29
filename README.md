# ProtechWithWAF #
Protecting web applications from common web vulnerablities, bots, and scrapers workshop

## **WARNING** ##
**In the lab several AWS resources will be created and bill the account the lab is provisioned in.  Please be sure to delete the cloudformation template and remove any resources created when you have completed the excersise to avoid any unwanted costs.**

The cloudformation template will create EC2 instances, NAT GW's, an ALB, WAF, Lambda's, an API GW, S3 Buckets, Kinesis Firehose, RDS Database, a Glue data catalog, and an ElasticSearch cluster.  The items created via the CloudFormation template should be removed when you delete the CloudFormation stack.  Any manual steps you take during the lab, in the AWS console or via the aws cli,  will need to be cleaned up.  At the end of each lab there will be instructions on how to clean up any of the manual steps performed.  If you have performed any additional steps on your own you should be sure to clean them up as well.

# Introduction #

In the labs you will get familar with AWS WAF and how you can use it protect a web application.

For lab1 you will perform some common web application attacks and then you will implement an AWS WAF to protect your application from these exposures without the need to update any web application code.

For lab2 you will use additional tools to create automations which will dynamically add AWS WAF protections from more advanced attacks.


# Requirements #
*   You will need to have the Microsoft Remote Desktop client installed on your local machine.  This is preinstalled on Windows systems.  If you are using MacOS you can install it from the Apple app store for free, but does require an Apple account to install.  For linux you can use <a href="http://www.rdesktop.org/">rdesktop</a>. For more information you can see the AWS documentation on all the <a href="https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/connecting_to_windows_instance.html#rdp-prereqs">prerequisites</a>.

# Known Issues #

*   When deploying the cloudformation template you can sometimes receive an error message when it's provisioning the AWS WAF. If this occurs you will need to delete the cloudformation stack and relaunch it. This is a timing related item which should be resolved for cloudformation in the future.

# Lab Documentation #

[Lab Prep](labPrep/README.md)

[Lab 1](lab1/README.md)

[Lab 2](lab2/README.md)

# Links #

Interesting links you may find useful to get additional information about AWS WAF automation.

*   <a href="https://aws.amazon.com/waf/">AWS WAF information page</a>
*   <a href="https://aws.amazon.com/solutions/aws-waf-security-automations/"> AWS WAF Security Automation Solution</a>
*   <a href="https://docs.aws.amazon.com/waf/latest/developerguide/tutorials-4xx-blocking.html"> Blocking IP Addresses that Submit Bad Requests</a>
*   <a href="https://github.com/awslabs/aws-waf-security-automations">Using bad actor IP blacklists to prevent web attacks</a>
*   <a href="https://aws.amazon.com/firewall-manager/">AWS Firewall Manager information page</a>
*   <a href="https://aws.amazon.com/blogs/security/how-to-analyze-aws-waf-logs-using-amazon-elasticsearch-service/">How to analyze AWS WAF logs using Amazon Elasticsearch Service</a>
*   <a href="https://aws.amazon.com/blogs/security/how-to-use-amazon-guardduty-and-aws-web-application-firewall-to-automatically-block-suspicious-hosts/">How to use Amazon GuardDuty and AWS Web Application Firewall to automatically block suspicious hosts</a>
