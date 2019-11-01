<!-- TOC -->

-   [Introduction](#introduction)
-   [Step 1: Associate WebCarter ALB with WAF-Automation Web ACL.](#step-1-associate-webcarter-alb-with-waf-automation-web-acl)
-   [Step 2: Configure ElasticSearch with Kibana and WAF full logging functionality](#step-2-configure-elasticsearch-with-kibana-and-waf-full-logging-functionality)
    -   [Configure Elasticsearch to consume WAF logs](#configure-elasticsearch-to-consume-waf-logs)
    -   [Configure Web ACL logging functionality.](#configure-web-acl-logging-functionality)
    -   [Upload dashboard and vizualizations to Kibana](#upload-dashboard-and-vizualizations-to-kibana)
-   [Step 3: Use ElasticSearch Service with Kibana for WAF monitoring and forensics](#step-3-use-elasticsearch-service-with-kibana-for-waf-monitoring-and-forensics)
-   [Step 4: Honeypot for bad bots and scrapers.](#step-4-honeypot-for-bad-bots-and-scrapers)
-   [Step 5: WAF Automation using AWS Lambda and Amazon Athena.](#step-5-waf-automation-using-aws-lambda-and-amazon-athena)
-   [Appendix:](#appendix)

<!-- /TOC -->
Lab 2 -- Automating Protections and Tracking Attacks
====================================================

### Introduction
In this WAF Automation lab we will use a Kinesis Firehose stream and setup ElasticSearch to consume the stream. We will configure the Web ACL WAF logs logging to the Kinesis Firehose stream.  We will use **Kibana** to do more analysis of our web traffic and help find requests which may not be legitimate.

Kinesis Data Firehose also places a backup copy of the WAF logs to an S3 bucket and we will Amazon Athena for additional analysis.

We will use a honeypot to detect bots crawler our websites and automate blocking these types of requests.

***

### Step 1: Associate WebCarter ALB with WAF-Automation Web ACL.

**Note**: Please see the [Appendix](#appendix) at the end if you are unable able to see the **WAF ACLs** in the **WAF & Shield** console.

1.  Go to AWS **WAF & Shield** console, choose the region you're working at, and open **waf-lab-1** Web ACL and navigate to the **Rules** tab on the right.

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image22.png" />

2.  In **AWS resources using this web ACL** delete association with WebCarter ALB.
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image21.png" />

3.  Choose WAF-automation Web ACL and open **Rules** tab.
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image23.png"  />

4.  Click **Add association**, then choose **Application load balancer** as **Resource type** and choose your WebCarter load balancer. Click **Add**.

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image24.png" />

    When you are done it should look similar to this.

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image25.png" />

Now your WebCarter Web application is protected with the Web ACL created for this lab.

***

### Step 2: Configure ElasticSearch with Kibana and WAF full logging functionality

Below you will create indexes and visualizations in your domain.

#### Configure Elasticsearch to consume WAF logs

**Note:** Please be sure to perform the steps in the order listed below when configuring Kibana.

1.  Go to ElasticSearch console, choose the domain created by waf-automation template, and click on the Kibana link:
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image1.png" />

2.  In the Kibana console
    *   Click on **DevTools** from the menu on the left -  Then click the button **Get to work** 	-  create an index using the following statement:

        ```json
        PUT _template/awswaf-logs
        {
           "index_patterns":[ "awswaf-*"],
           "settings":{
                "number_of_shards":1
           },
           "mappings":{
              "waflog":{
                 "properties":{
                    "httpRequest":{
                       "properties":{
                          "clientIp":{
                             "type":"keyword",
                             "fields":{
                                "keyword":{
                                   "type":"ip"
                                }
                             }
                          }
                       }
                    },
                    "timestamp":{
                       "type":"date",
                       "format":"epoch_millis"
                    }
                 }
              }
           }
        }
        ```

    *   Paste the statement in Kibana console and click on the green **play** button to execute it:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image2.png" />

#### Configure Web ACL logging functionality.

3.  In the **WAF & Shield** AWS console, open **Logging** tab of the **mod-module-1** Web ACL and click **Enable Logging**.
    **Note**: If you see the splash/informational screen when opening the **WAF & Shield** console please see the [Appendix](#appendix)

4.  Choose Kinesis Data Firehose stream created for the lab **aws-waf-mod_module-1**.

5.  **Do Not** add **Redacted Fields** and click **Create**.

6.  RDP to the Attacker Windows instance and run **BOT1** Jmeter scenario using the BOT1 shortcut on the desktop. Click the green play button in the top menu ribbon to generate data flowing to Kinesis Data Firehose and then to ElasticSearch domain.

7.  Wait for a couple of minutes till WAF logs get to the ElasticSearch domain, then in the **Kibana** dashboard click on the **Discover** menu and define an index pattern as *awswaf-\** and click **Next step**:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image3.png" />

8.  Choose timestamp as a Filter and click **Create index pattern**.
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image4.png" />

9.  Now you're able to run searches through your logs by going into the **Discover** tab in **Kibana**. For example, you can look for specific HTTP headers, query strings, or source IP addresses:

#### Upload dashboard and vizualizations to Kibana

>You can always customize them or create new visualizations as required.

10. Go to **Management** menu and click **Advanced**. Search for the the ***defaultindex*** then copy and save the default index id value:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image5.png" />

11. Download and using the URL in the JAM **Output Properties** webpage or you can right click the *kibana-config.json* url and select save link as. Open kibana-configruation.json locally with a text editor and replace the ```_YOUR_ES_INDEX_``` value with the one saved above. Please do it everywhere in the file (approximately 5 times).
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image26.png" />

12. Save the file.

13. Go to **Management -> Saved Objects** tab, click **Import** and upload the modified kibana-configruation.json file. Click **Yes** for the overwrite question.

14. You'll see your dashboard and visualizations have been uploaded successfully:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image6.png" />

15. Navigate to the **Dashboard** menu and choose **WAF** dashboard you just uploaded.

***

### Step 3: Use ElasticSearch Service with Kibana for WAF monitoring and forensics

1.  On the Attacker Windows instance, check if **BOT1** Jmeter scenario is still running. Re-start it if needed to generate more data for this exercise.

2.  Go to AWS Console and choose **ElasticSearch** service. Choose your **mod-module-1-awswaf-logs** domain and click on the Kibana link.

3.  Navigate to the **Dashboard** menu on the left and explore the dashboard graphs. Check the number of requests per minute per source IP address.

4.  Go to AWS **WAF & Shield** console, choose **IP addresses**, from the left side menu, check if IP addresses you see in the Kibana charts appear in any of the block lists.

**QUESTION**: *Should the source IP addresses for BOT1 scenario appear in any of the block lists?*

This scenario simulates a mix of legitimate traffic together with under the radar BOT traffic.

5.  Go back to Kibana **Dashboard** GUI and scroll down to **Allow/block URIs**. Note that /login page is one of the top requested URIs.

6.  Go to **Discover** menu and enter the following search filter:
    ```
    httpRequest.uri:/user/login
    ```

7.  Scroll through some of the requests and note that at glance, they're coming from the same source IP address.

8.  Enter the following search filter.
    ```
    !httpRequest.uri:/user/login && httpRequest.clientIp:10.192.30.50
    ```

**QUESTION**: *Do these requests look legit? In the real-life scenario why can we see large number of malicious and legitimate requests coming from the same IP address?*

*Can we just block all requests coming from that IP address?*

*Can we block all requests to this URI?*

9.  Look at the User-Agent header value for the requests returned by the search below:
    ```
    httpRequest.uri:/user/login && httpRequest.clientIp:10.192.30.50
    ```

    **Note** that large number of requests has User-Agent value **IamaBOT**. We'll create a WAF rule that blocks these requests below.

10. Go to AWS WAF & Shield console and navigate to **String and regex matching**.

11. Click **Create condition** and create BOT10 condition as following:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image7.png" />

12. Click **Add filter** then **Create**.

13. Go to **Rules** menu and click **Create rule**.

14. Create **BOT10** rule as following:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image8.png" />

15. Click **Create**.

16. Navigate to **Web ACLs** menu and click on **mod-module-1** ACL.

17. Click **Rules** tab then **Edit Web ACL** button.

18. In the **Rules** drop-down list choose BOT10 rule and click **Add rule to Web ACL**.

19. Assure rule action set to **Block** and click **Update**.

20. Wait for couple for minutes and refresh Kibana dashboard. Check if you see blocked request to the login page.

    **Note** the **Terminating Rule ID** in the **WAF Rules Hit** visualization and check the rule that is blocking these requests. You can check the rule by going to the **WAF & Shield Console**, filter on the appropriate region and select **Rules** from the left side.  This should display a list of all the rules and ID of each rule.  The **Terminating Rule ID** should match the rule you recently defined.

    >You can also run the following AWS CLI command from a machine which has the AWS CLI installed, credentials with the appropriate permissions, and internet access. Update the region and the rule id as required:
    >
    >Example:
    >```sh
    >aws waf-regional get-rule --region us-west-2 --rule-id d3458c9b-5bba-4361-a7ce-e28d66a86d27
    >```
    >Run the following replacing items in "<>":
    >```sh
    >aws waf-regional get-rule --region <Region> --rule-id <Terminating Rule ID>
    >```


**QUESTION**: Does number of allowed requests to the login page look reasonable after we blocked a **IamABOT** BOT?

22. Enter the following filter in the **Kibana console -> Discover** and try to see if there is anything abnormal in the requests returned:
    ```
    action:"ALLOW" && httpRequest.uri:"/user/login"
    ```

**QUESTION**: If we cannot find how to identify malicious requests and we cannot block this source IP entirely, what can be next steps of our investigation?

In the next steps we'll look at ALB access logs using Amazon **Athena**.

23. Go to Amazon **Athena** AWS console.

24. Go to Query Editor and choose **mod-module-1_..** as your
    Database.

    **Note:** You will need to supply a s3 bucket location to store the query editor output before you are able to run a query.  You can get the location information from the JAM **Output Properties** webpage.

    1.  Click **setup a query result location in Amazon S3**.
        <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image27.png" />

    2.  Copy the bucket information from the **Output Properties**: **AthenaEditorOutput**.
        <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image31.png" />
    3.  Paste the copied bucket location in the **Query result location** and then click **Save**.
        <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image30.png" />

25. Run the following SQL query:
    ```sql
    SELECT * from app_access_logs limit 1;
    ```

    **app_access_logs** query points to the ALB access logs on the corresponding S3 bucket.

    You can get an additional information about incoming requests from these logs, such as HTTP response code, SSL cypher and protocol, processing time. This information usually helps you to investigate incoming requests to your Web application.

26. Get familiar with the values returned.

27. Run the following SQL query:

    ```sql
    SELECT client_ip,
    			request_url,
    			target_status_code,
    			COUNT(*) AS counter
    FROM app_access_logs
    WHERE parse_datetime(time, 'yyyy-MM-dd''T''HH:mm:ss.SSSSSS''Z') > DATE_ADD('minute', -5, NOW())
		    	AND target_status_code = ANY (VALUES '400', '401', '403', '404', '405', '500')
    GROUP BY client_ip, request_url, date_trunc('minute',
      parse_datetime(time, 'yyyy-MM-dd''T''HH:mm:ss.SSSSSS''Z')),
      target_status_code;
    ```

You should see there are malicious requests coming from the same IP address and targeting our Web application REST API endpoint. We know that REST API shall be open only to our partners, and we decide quickly to block it for the IP address returned above.

28. Go to AWS **WAF & Shield** console and navigate to **String and regex matching**.

29. Click **Create condition** and create RESTAPI condition as following:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image9.png" />

30. Click **Add filter** then **Create**.

31. Go to **IP addresses** and click **Create condition**:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image10.png" />

32. Click **Add IP address...** then **Create**.

33. Go to the **Rules** menu and click **Create rule**:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image11.png" />
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image12.png" />

    **Note** we added two conditions in this rule.

34. Navigate to **Web ACLs** menu and click on **mod-module-1** ACL.

35. Click **Rules** tab then **Edit Web ACL** button.

36. In the **Rules** drop-down list choose RestAPIRule rule and click **Add rule to Web ACL**.

37. Assure rule action set to **Block** and click **Update**.

38. Wait for couple for minutes and refresh your Kibana dashboard. Check if you see blocked request to the api endpoint.

39. Go to AWS **Athena** console and run the following query:

    ```sql
    SELECT client_ip,
    			request_url,
    			target_status_code,
    			COUNT(*) AS counter
    FROM app_access_logs
    WHERE parse_datetime(time, 'yyyy-MM-dd''T''HH:mm:ss.SSSSSS''Z') > DATE_ADD('minute', -5, NOW())
    			AND elb_status_code = '403'
    GROUP BY client_ip, request_url, date_trunc('minute',
       parse_datetime(time, 'yyyy-MM-dd''T''HH:mm:ss.SSSSSS''Z')),
       target_status_code;
    ```

You can see that requests from that IP address to /api url been blocked by the WAF rule.

***

### Step 4: Honeypot for bad bots and scrapers.

1.  Copy **AddHoneypotSsmCommand** from the JAM **Output Properties** webpage.

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/RunCommand.png" />

2.  Paste into the Command Prompt inside the Attacker Windows instance.  You can open a command prompt by using the **CMD** shortcut on the desktop.

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/RunCommand-cmd.png" />

3.  Wait a few moments

4.  Open Chrome browser and navigate to the WebCarter home page.

5.  Right click on the page and choose **Inspect**.

6.  Verify the honeypot link is present on the page, right click on the page and select **View Page Source** and search for honeypot. This link has been inserted into your application by AWS **Systems Manager**, which was configured by JAM event.
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image13.png" />

    An API Gateway instance has been preconfigured with a Lambda function that parses the source IP address of the request and adds the source IP address to the black list.

7.  To review / customize the honeypot lambda function, go to AWS Console and choose **API Gateway** service.

8.  Choose **Security Automation -- WAF..** API and click **ANY**.

9.  Click on the Lambda function on the right if you'd like to customize this function:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image14.png" />

10. Run **HTTTRack** web copier on your Windows instance from the shortcut on the desktop.

11. Select your language and click **Next** on the Welcome page and enter your project name (WebCarter for example), then click **Next**:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image15.png" />

12. Click **Add URL**, enter the ALB endpoint and click **OK**:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image16.png" />

13. Click **Set options** button and go to **Experts Only** tab. Change **Global travel mode** to **Go everywhere...** and click **OK**. Because the honeypot is a different domain, this will allow us to simulate a bad bot hitting the **no-follow** hidden honeypot link above:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image17.png" />

14. Click **Next** then **Finish**.

15. You can cancel the test after you see the site been downloading. It was enough time for the copier to follow the honeypot URL.

16. Navigate to WAF& Shield AWS Console, **IP Addresses** and choose **..Bad Bot Set**. Assure you see your external IP address in the list.

In this simulation HTTTrack is accessing an ALB using 10.x.x.x IP address, while accessing API GW honeypot endpoint using public IP address. Due to this simulation network topology, we can see our public source IP address has been added to the block list by the Lambda function, although traffic from HTTTrack is not been blocked. In the real-life scenario scraper will be blocked based on its public IP address.

***

### Step 5: WAF Automation using AWS Lambda and Amazon Athena.

In the above steps we demonstrated manual creation of the WAF rules using information we've got from Kibana dashboard and Athena SQL queries. Below we'll work with automated protection configured the set up of the lab.

1.  Go to WAF & Shield AWS Console, IP Addresses and click on **..Scanners & Probes Set**.

2.  Assure the list is currently empty.

3.  Go to Amazon Athena console and navigate to **Saved Queries** tab.

4.  Click on **ScannersProbesLogParser** query.

5.  For our exercise, change the time window from 240 minutes to 5 minutes:
    ```sql
    DATE_ADD('minute', -5, NOW())
    ```
    Now this modified query will be looking for any IP address that got \>=50 of the 4xx responses from the application within last 5 minutes.

6.  Click **Save as** and save the modified function with the new name **ScannersProbesLogParser5min**.

7.  In AWS CLI run the following command updating the region as required (here and in all commands below):
    ```sh
    aws athena list-named-queries --region us-west-2
    ```
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image18.png" />


8.  Copy the id of the first query in the list and run the following command with that query id:
    ```sh
    aws athena get-named-query --region us-west-2 --named-query-id 2ff43345-277d-4614-8020-b76f56e02757
    ```
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image19.png" />

9.  Check the query name and assure this is the query we just saved on the console.

10. Go to AWS Lambda console and search for **..LambdaLogParserFunction..**

11. Open **..LambdaLogParserFunction..** and click on the **CloudWatch Events**.

12. Click on **...LambdaAthenaAppLogParser...** one.

    **Note** If you don't see it scroll down further on the page.

13. In the CloudWatch window click **Action-\>Edit** on the top right corner.

14. In the JSON input definition scroll right and replace the query id with the one you saved above:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image20.png" />

15. Click **Configure details** and then **Update rule**.

16. Run **BotsScrappers** scenario on JMeter using the **BotsScrapp..** shortcut on the desktop and click the green play button in the top menu ribbon.

17. Go to the results tree in JMeter and verify that application returns 404 **Not found** response.

18. After about 5 minutes go to AWS WAF & Shield console, navigate to **IP addresses** and click on **...Scanners & Probes Set**. Verify that your source IP address appeared in the blocking list.

19. Go to JMeter results tree, clear the results and check that currently JMeter is getting HTTP 403 **Forbidden** response from WAF.

20. Stop JMeter test and check that in about 5 minutes your source IP address will disappear from the **...Scanners & Probes Set** IP list.

In this scenario we used automated detection of probes and scanners, that are getting 4xx responses from the application and stay undetected by the rate-limiting rules.

***

### Appendix:

*   When opening the AWS WAF & Shield console you maybe prompted with the WAF & Shield splash screen instead WAF specific console.

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image28.png" />
*   Click the **Go to AWS WAF** button
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image29.png" />
*   Then click **Web ACLs**
*   Update the **Filter** to reflect the appropriate region where the ALB is deployed.

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab2/media/image34.png" />
