Pre-requisites
==============
-   A computer with Chrome
-   A Remote Desktop (RDP) client
-   An [AWS EC2 Key Pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair)
-   Optional - git

Lab Prep -- Launch Cloudformation Stacks Setup
==============================================

(Estimated completion time -- 35 minutes)

Lab preparation will take some time to complete, so we recommend that
you initiate this and let it complete in the background.

### Web Carter & Attacker workstation setup


1.  Open the Cloudformation console

    ![](.//media/image10.png)

2.  Create a new stack

    ![](.//media/image11.png)

3.  Upload stack template **template/aws-waf-security-automations-template.yaml**

    ![](.//media/image12.png)

4.  Enter a stack name in all lowercase letters -- e.g. "waflab"

    ![](.//media/image13.png)

5.  Get your IP from Amazon checkip -> <http://checkip.amazonaws.com>

    ![](.//media/image7.png)


6.  At the bottom of the Environmental Configurations page enter the following.

    a.  **AdminPassword** A password at least 8 characters long and should include at least 3 of the 4 types of characters, lowercase, uppercase, number, and special character.
    **Note** you will likely not need this password for the lab.

    b.  **KeyName** An AWS EC2 key pair
    -   This list is generated from **EC2/keypairs** in the EC2 console for the region you are launching the stack in.  You must have this pem file or the data from this file to gain access to the windows instance.

    c.  **MyIP** The IP address or range of IP's and the subnet mask you will be connecting from.
    -   i.e. single ip would be 192.168.10.5/32, class C range would be 192.168.10.0/24.  These examples are private IP's you would need to use the ip returned from the checkip url.

    ![](.//media/image14.png)

7.  Scroll to the bottom of the page and hit **Next**

8.  Scroll to the bottom of the page and hit **Next** on *Configure
    stack options* page

9.  Scroll to the bottom of the page and check **I acknowledge that AWS CloudFormation might create IAM resources.**, the click **Create**

    ![](.//media/image16.png)

10. The stack will eventually turn to CREATE\_COMPLETE.

11. Go to the stack’s **Output**.

    ![](.//media/image17.png)
