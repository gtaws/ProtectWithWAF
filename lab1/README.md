Lab 1 -- Manually testing attacks and protections
=================================================

###Preparation: Signup for AWS Jam events and launch the lab

1. Go to the following URL [https://jam.awsevents.com](https://jam.awsevents.com)
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/jam1.png" width="700" />

2. Click the [**Register**](https://jam.awsevents.com/register) link in the top right corner.  
**Note:** If you have an account and are logged in you can select the Join a Jam from the menu option in the top right or you can go to the following link [Join a Jam](https://jam.awsevents.com/home/join) and enter the secret key provided in the workshop, then proceed to step 6.

3. Supply the requested information (email address, displayname, password) and click submit. You will need to have access to the email account you register with.

4. You will then be prompted to supply a verification code which was sent to the email address you used in step 3.  Enter the code and click **Confirm**. If you do not see the verifcation email right away please check your spam/junk folder.

5. You will then be prompted to [Join a Jam](https://jam.awsevents.com/home/join).  Enter the secret key provided in the workshop.   
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/jam2.png" width="700" />

6. Download the SSH Key Pair by selecting **AWS account** on the left side and select the SSH Key Pair **Download** button.  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/jam3.png" width="700" />

7. In **Output Properties** launch the link for **UpdateAttakerSecurityGroup**. This will determine your internet IP address and add the appropriate security to allow access to resources we will be using in the lab.  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/jam5.png" width="700" />
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/jam6.png" width="700" />

8. While in **Output Properties** copy and save the ALB name and ALB URL.  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/jam5.png" width="700" />
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image32.png" width="700" />
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image33.png" width="700" />

***

###Step 1: Simulate Cross-Site Scripting, SQL Injection, HTTP Flood, and Bad BOT activities####

####Introduction:
You will be using this Windows instance to simulate web application attacks using the following tools, which have been pre-installed on the Windows Attacker instance, which you will be using throughout lab.  

- Chrome Browser
- Apache JMeter: <https://jmeter.apache.org/>
- Apache Benchmark (Ab):
<https://httpd.apache.org/docs/2.4/programs/ab.html>
- HTTRack: <https://www.httrack.com/>

You will also be using AWS console, to access you must use the link supplied in the AWS Jam events web page, selecting the **Open AWS Console** button.

1.  RDP (Remote Desktop Protocol) to the Attacker windows instance.  
**Note:** You can find the public ip of the Attacker windows instance in the **Output Properties** section on the Jam event web page and here are instructions on [how RDP to ec2 instance](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/connecting_to_windows_instance.html).  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/jam8.png" width="700" />
 - **Note:** Once you login you may be prompted to allow your computer to be discoverable.  You can select **No**, but either will be fine.

2.  On the Attacker instance (RDP Session) open Chrome browser and navigate to the ALB endpoint you saved above:
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image2.png" width="700" />

3.  Right click on the page and choose **Inspect**. Chrome Developer Tools will open.  

4.  In Chrome Developer Tools go to **Network** tab.  

5. In the browser window on the WebCarter site type the following in the **Search** text field and click **Search!**:
```<script>alert(document.cookie)</script>```

6. In Chrome Developer tools select the **search?id...** request in the Network tab and navigate to Response sub tab on the right. If you do not see **search?id..** then Click on the first green bar field.  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/chromenetwork.png" width="700" />

7.  Use CTRL-F combination to search for **alert** in the response. As far as you can see, server returned your input above without validating it:  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image3.png" width="700" />


8.  Click on any of the Brands filters (Apple, NBA). The script will be executed and the session cookie will be printed in the alert. Your Cross-site scripting attack was successful:  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image4.png" width="700" />


9.  Click **Clear** button to clear the Developer Tools output:  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image5.png" width="700" />

10. In the **Search** field enter the following:  
> *' and 1=0 union select 1,group\_concat(table\_name) from
> information\_schema.tables \#*

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image6.png" width="700" />

11. Server returns **Database Error** page displaying the SQL statement been executed. You started SQL Injection attack and got a desired output from the server. You can continue to explore SQL Injection vulnerabilities of WebCarter application later on.  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image7.png" width="700" />

12. Run Apache JMeter by clicking on the JMeter shortcut on the desktop.  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image8.png" width="700" />

13. Go to **File -\> Open** menu and navigate to **c:\\JmeterScenarios.jmx** file and click Open:  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image9.png" width="700" />


14. Click the most top **HTTP Requests Defaults** entry in the test plan menu. Update the **Server Name or IP** with the ALB endpoint saved in the p. 1 above. Enter the server name only without without the **"/"** suffuxess:  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image10.png" width="700" />

15. Save changes by navigating to **File-\>Save** menu.

16. Right click on the **DoS** scenario and then click **Start**:  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image11.png" width="700" />

17. Navigate to **View Results Tree** and assure you're getting 200 OK responses. Your attack was successful:  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image12.png" width="700" />

18. Clear the results tree for the next tests:  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image13.png" width="700" />

19. Open a cmd window and change directory to **C:\Apache24\bin**.  

20. Run Apache Benchmark:
    -  Copy the abLoadTestCommand from the **Output Properties** tab on the AWS Jam events web page.  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/jam7.png" width="700" />  
    -   Run the command in a Command Prompt window  
    -   Assure that ab run above was successful:  

> *Concurrency Level: 1*
>
> *Time taken for tests: 6.813 seconds*
>
> *Complete requests: 100*
>
> *Failed requests: 0*
>

In this test we simulated a simple bot that causes excessive load on the server and can be identified by its *User-Agent.*
    

***

**Step 2: Configure protection against the above attacks using WAF
native rules.**  

1.  Login to AWS console and choose EC2 service.

2.  Choose **Load Balancers** in the left menu and then choose your load balancer created by CloudFormation stack.  

3.  Go to **Integrated Services** tab and click on **Create Web ACL** button. You'll be directed to the WAF & Shield service console.  

4.  Choose the region you're using in the **Filter** field:  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image15.png" width="700" />
**Note**: WAF configuration is separate for each region and separate for **Global** deployment used for CloudFront deployments. You must configure
your Web ACL in the same region where you have your ALB deployed.

5.  Click **Create web ACL** and fill out the details as below and click **Next**:
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image16.png" width="700" />

6.  Create Cross-site scripting match condition. Click **Add Filter** then **Create**.
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image17.png" width="700" />
**Note**: Filters in the conditions are logically disjoined, i.e. **OR**.
You can have up to 10 filters in each condition (hard limit).

7.  Create SQL injection match condition. Click **Add Filter** then **Create**.
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image18.png" width="700" />

8.  Create String and regex match condition. Click **Add Filter** then **Create**.
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image19.png" width="700" />

9.  Click **Next** on the *Create Conditions* page.

10. Click **Create Rule** button.

**Note:** Be sure to pick the appropriate match condition for each of the rules you are creating.  For example the **XSS1** condition you would want to select the **match at least one of the filters in the cross-site scripting match condition**.  In the screenshots below each rule include the appropriate match condition.

11. Create a simple Cross-site scripting rule that in our example will contain a single XSS condition created above. Select the condition **XSS1** as below and click **Add Condition**:
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image20.png" width="700" />
**Note**: Conditions in the rule are logically joined, i.e. **AND**. You can have up to 100 conditions of each type per account (soft limit).

12. Create SQL Injection rule choosing the SQL injection condition created above:
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image21.png" width="700" />

13. Create Bad Bot rule choosing the String match condition created above:
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image22.png" width="700" />

14.  Create HTTP Flood rule changing the Rule type to **Rate-based rule**:
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image23.png" width="700" />
**Note**: No conditions are needed for this rule, but you could add match conditions to the rate-based rule when crafting your own custom signatures.

15. Set default Web ACL action as **Allow all requests that don't match any rules** and click **Review and create**.

16. Review the Web ACL and click **Confirm and create**.

**Note**: Because we started creating a Web ACL from the ALB console, your Web ACL is already associated with your load balancer. You also can just create Web ACL and then associate it with one or more load balancers.

*** 

**Step 3: Demonstrate protection against the above attacks using WAF
native rules.**

In this step we'll repeat the attacks in Step 1 above and assure they're blocked by the rules we created in Step 2.

1.  In a Chrome browser window navigate to your ALB endpoint and type following in the **Search** string.  Then click **Search!**. ```<script>alert(document.cookie)</script>```

2.  Assure you get HTTP 403 response from WAF:
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image24.png" width="700" />

3.  In the **Search** field enter the following:
> *\' and 1=0 union select 1,group\_concat(table\_name) from
> information\_schema.tables \#*

4.  Assure you get HTTP 403 response from WAF:
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image24.png" width="700" />

5.  Start JMeter DoS scenario and navigate to the Results tree. Watch the request till they start failing after reaching 2000 requests / 5 min as configured in your rate-based rule:
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image25.png" width="700" />

6.  Go to AWS WAF Console and navigate to Rules-\> HTTPFlood1Rule.  Notice IP addresses that are currently blocked by this rule:
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image26.png" width="700" />
These IP addresses configured in JMeter in **CSV Data Set Config** section and originate from the attacker instance.

7.  Repeat ab test as above and notice Non-2xx responses from WAF:
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image27.png" width="700" />

In this lab you protected your Web application against Cross-site scripting, SQL Injection, HTTP flood and simple BOTs using AWS WAF native rules.

***

###Appendix:
- When opening the AWS WAF & Shield console you maybe prompted with the WAF & Shield splash screen instead WAF specific console.  
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image28.png" width="700" />   
- Click the **Go to AWS WAF** button
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image29.png" width="700" />  
- Then click **Web ACLs**
- Update the **Filter** to reflect the appropriate region.  This is defined on the JAM AWS webpage listed under **AWS Account -> AWS Region**.  
- <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image34.png" width="700" />