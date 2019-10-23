Lab 1 -- Manually testing attacks and protections
=================================================

### Introduction:
You will be using this Windows instance to simulate web application attacks using the following tools, which have been pre-installed on the Windows Attacker instance, which you will be using throughout lab.
*   Chrome Browser
*   <a href="https://jmeter.apache.org/">Apache JMeter</a>
*   <a href="https://httpd.apache.org/docs/2.4/programs/ab.html">Apache Benchmark (Ab)</a>
*   <a href="https://www.httrack.com/">HTTRack</a>

Then you will be creating an AWS WAF to protect against these simulated attacks and validating the web application is protected by these attacks.

***

### Preparation:
You will be using Windows Remote Desktop (RDP - Remote Desktop Protocol) to connect to an EC2 windows instance in AWS.  You will need to have a remote desktop client installed to procceed with the lab.  If you are not formiliar with how to connect to an EC2 instance I would suggest reviewing the full instructions on the AWS website <a href="https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/connecting_to_windows_instance.html#rdp-prereqs">Connecting to Your Windows Instance</a>.  Here is a summary of the steps you will need to perform to connect to the Attacker instance throughout the lab.

1.  You will need your EC2 keypair pem file (\<keypair filename\>.pem), which was used to create these instances.
2.  Login to the **AWS console** and choose the **EC2** service, typing *EC2* in the **Find Services** search is one way to get to the EC2 console.
3.  Be sure to select the appropriate region from the top right drop down box.  This should match where the lab resources are deployed.
4.  From the menu on the left choose **Instances**
5.  Select the **Attacker** instance by selecting the box to the left of the instance name.  This should be the only instance selected.
6.  Near the top of the EC2 console select the **Connect** button
7.  Select the **Get Password** button
8.  Select **Choose File** and supply the keypair pem file
9.  Select **Decrypt Password**
10. Hover your mouse just over to the right of the password displayed and you should see an icon which when clicked will copy the password.
11. Select **Download Remote Desktop File** to download and then run it.
12.You should be prompted for user credentials and the user name should be prefilled with **Administrator** and you can paste in the password copied above.  Once you have supplied the ID/Password click the appropriate button to proceed.
    *   If you have the option to save the password feel free to check that box before connecting.
    *   If you are not prompted for credentials it's likely the security group on your instance is not allowing your source IP address.  Please make sure you have completed all of preperation steps and you can also review your security group rules assigned to the ec2 instance.

***

### Step 1: Simulate Cross-Site Scripting, SQL Injection, HTTP Flood, and Bad BOT activities

#### Simulate a cross site scripting attack.
1.  Remote desktop (RDP) to the **Attacker** windows instance, using the Preperation steps above.
    *   **Note:** Once you login you may be prompted to allow your computer to be discoverable.  You can select **No**, but either will be fine.

2.  On the Attacker instance (RDP Session) open Chrome browser and navigate to the ALB endpoint you saved above:
![](.//media/image2.png)

3.  Right click on the page and choose **Inspect**. Chrome Developer Tools will open.

4.  In Chrome Developer Tools go to **Network** tab.

5.  In the browser window on the WebCarter site type the following in the **Search** text field and click **Search!**:
```
<script>alert(document.cookie)</script>
```

6.  In Chrome Developer tools select the **search?id...** request in the Network tab and navigate to Response sub tab on the right. If you do not see **search?id..** then Click on the first green bar field.
![](.//media/chromenetwork.png)

7.  Use CTRL-F combination to search for **alert** in the response. As far as you can see, server returned your input above without validating it:
![](.//media/image5.png)

8.  Click on any of the Brands filters (**Apple**, **NBA**). The script will be executed and the session cookie will be printed in the alert. Your Cross-site scripting attack was successful:
![](.//media/image4.png)

9.  Click **Clear** button to clear the Developer Tools output:
![](.//media/image3.png)

#### Simulate a SQL injection attack.

10. In the **Search** field enter the following and click **Search!**:
```
' and 1=0 union select 1,group\_concat(table\_name) from information\_schema.tables \#
```
![](.//media/image6.png)

11. Server returns **Database Error** page displaying the SQL statement been executed. You successfully performed a SQL Injection attack and got a desired output from the server.
![](.//media/image7.png)

#### Simulate a HTTP Flood attack

12. Run Apache JMeter by clicking on the JMeter shortcut on the desktop.

    ![](.//media/image8.png)

13. Go to **File -\> Open** menu and navigate to **c:\\JmeterScenarios.jmx** file and click Open:
![](.//media/image9.png)

14. Click the most top **HTTP Requests Defaults** entry in the test plan menu on the left. Update the **Server Name or IP** with the ALB endpoint saved. Enter the server name only without without **"https://"** prefixes or **"/"** suffuxes:
![](.//media/image10.png)

15. Save changes by navigating to **File-\>Save** menu.

16. Right click on the **DoS** scenario and then click **Start**:
![](.//media/image11.png)

17. Navigate to **View Results Tree**, select one of the items which has one of the item with a green shield under, and assure you're getting 200 OK responses. Your HTTP Flood attack was successful:
![](.//media/image12.png)

18. Stop and Clear the results tree for the next tests:
![](.//media/image13.png)

#### Simulate a Bad Bot attack
19. Open a cmd window and change directory to **C:\Apache24\bin**.

20. Run Apache Benchmark:
    -  Copy the abLoadTestCommand from the **Outputs** tab on of the CloudFormation stack.
![](.//media/image14.png)
	  -  Run the command in a Command Prompt window
	  -  Assure that ab run above was successful:

> *Concurrency Level: 1*
>
> *Time taken for tests: x.xxx seconds*
>
> *Complete requests: 100*
>
> *Failed requests: 0*
>

21. In this test we simulated a simple bot that causes excessive load on the server and can be identified by its *User-Agent.*

***

### Step 2: Create a WAF and configure protection against the above attacks using WAF native rules.

1.  Login to AWS console and choose EC2 service.

2.  Choose **Load Balancers** in the left menu and then choose your load balancer created by CloudFormation stack.

3.  Go to **Integrated Services** tab and click on **Create Web ACL** button. You'll be directed to the WAF & Shield service console.

4.  Choose the region you're using in the **Filter** field:
![](.//media/image15.png)
  **Note**: WAF configuration is separate for each region and Global for CloudFront deployments. You must configure your Web ACL in the same region where you have your ALB deployed.

5.  Click **Create web ACL**, fill out the details as below, and click **Next**:

    **Note**: Please select the region the ALB is deployed.
    ![](.//media/image16.png)

6.  Create Cross-site scripting match condition. Click **Add Filter** then **Create**.
![](.//media/image17.png)
    **Note**: Filters in the conditions are logically disjoined, i.e. **OR**.
You can have up to 10 filters in each condition (hard limit).

7.  Create SQL injection match condition. Click **Add Filter** then **Create**.
![](.//media/image18.png)

8.  Create String and regex match condition. Click **Add Filter** then **Create**.
![](.//media/image19.png)

9.  Click **Next** on the *Create Conditions* page.

10. Click **Create Rule** button.

**Note:** Be sure to pick the appropriate match condition for each of the rules you are creating.  For example the **XSS1** condition you would want to select the **match at least one of the filters in the cross-site scripting match condition**.  In the screenshots below each rule include the appropriate match condition.

11. Create a simple Cross-site scripting rule that in our example will contain a single XSS condition created above. Select the condition **XSS1** as below and click **Add Condition**:
![](.//media/image20.png)
    **Note**: Conditions in the rule are logically joined, i.e. **AND**. You can have up to 100 conditions of each type per account (soft limit).

12. Create SQL Injection rule choosing the SQL injection condition created above:
![](.//media/image21.png)

13. Create Bad Bot rule choosing the String match condition created above:
![](.//media/image22.png)

14. Create HTTP Flood rule changing the Rule type to **Rate-based rule**:
![](.//media/image23.png)
**Note**: No conditions are needed for this rule, but you could add match conditions to the rate-based rule when crafting your own custom signatures.

15. Set default Web ACL action as **Allow all requests that don't match any rules** and click **Review and create**.

16. Review the Web ACL and click **Confirm and create**.

**Note**: Because we started creating a Web ACL from the ALB console, your Web ACL is already associated with your load balancer. You also can just create Web ACL and then associate it with one or more load balancers.

***

### Step 3: Demonstrate protection against the above attacks using WAF native rules.

In this step we'll repeat the attacks in Step 1 above and assure they're blocked by the rules we created in Step 2.

1.  In a Chrome browser window navigate to your ALB endpoint and type following in the **Search** string.  Then click **Search!**.

    ```<script>alert(document.cookie)</script>```

2.  Assure you get HTTP 403 response from WAF:
![](.//media/image24.png)

3.  In the **Search** field enter the following:
```
' and 1=0 union select 1,group\_concat(table\_name) from information\_schema.tables #
```

4.  Assure you get HTTP 403 response from WAF:
![](.//media/image24.png)

5.  Start JMeter DoS scenario and navigate to the Results tree. Watch the request till they start failing after reaching 2000 requests / 5 min as configured in your rate-based rule:
![](.//media/image25.png)

6.  Go to AWS WAF Console and navigate to Rules-\> HTTPFlood1Rule.  Notice IP addresses that are currently blocked by this rule:

    ![](.//media/image26.png)
    These IP addresses configured in JMeter in **CSV Data Set Config** section and originate from the attacker instance.

7.  Repeat ab test as above and notice Non-2xx responses from WAF:
![](.//media/image27.png)

In this lab you protected your Web application against Cross-site scripting, SQL Injection, HTTP flood and simple BOTs using AWS WAF native rules.

***

### Cleanup
1.  Login to AWS console and choose **WAF & Shield** service.

2.  In the **Filter** section select the appropriate region you created the WAF in.

3.  In the **Web ACLs** section.  Click on the WAF name you created and select the **Rules** tab. If you followed name listed in the lab the name will be **waf-lab-1**.

4.  Under **AWS resources using web ACL** (bottom) click the **X** icon to remove the ACL from the Application load balancer.

5.  Select the **Edit web ACL** button (top right of the **Rules** tab).  On the **Edit web ACL \<waf name\>** click the **X** icons for each of the rules attached to the web ACL until there are no rules remaining.  Then select **Update**.

6.  You should be back at the **Web ACLs** page.  Make sure the WAF you created is selected still and click the delete button.

7.  Click Rules from the left hand side.  You should see the 4 rules BadBot1Rule, HTTPFlood1Rule, SQLInjection1Rule, and XSS1Rule if you followed the naming in the lab.

8.  Select HTTPFlood1Rule and select the **Delete** button.  You will be prompted with a warning, Click the **Delete** button.

9.  For the remaining rules you will need to click on the rule name, example select BadBot1Rule, in the right pane
    1.  Click **Edit rule**.
    2.  Select the **X** in the top right of the defined condition filter.
    3.  Select update.  You should get the message **All conditions for this rule will be removed** in a yellow box.
    4.  Once you have removed all matched conditions for the the rule you can then delete the rule by selecting the radio button and then clicking the **Delete** button.  You will get a warning again, click the **Delete** button.

10. Perform the steps outlined in step 9 above for each of the remaining rules (SQLInjection1Rule & XSS1Rule)

***

### Appendix:
*   When opening the AWS WAF & Shield console you maybe prompted with the WAF & Shield splash screen instead WAF specific console.

    ![](.//media/image28.png)
*   Click the **Go to AWS WAF** button
![](.//media/image29.png)
*   Then click **Web ACLs**
*   Update the **Filter** to reflect the appropriate region where the ALB is deployed.

    ![](.//media/image34.png)
