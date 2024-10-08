# TA-WA Optimization - starter

Author / Contributor
* Stephen Salim
* Carlos Perez


![TA-WA-Optimizer-HighArch.png](./static/images/TA-WA-Optimizer-HighArch.png)


## What does this sample automation do ?

This sample solution's objective is to aid Cloud optimization review, using Well-Architected framework. 

The solution compiles a list of resource configurations, that are not aligned with AWS known best practices, leveraging AWS Trusted Advisor.

Below is the detailed solution's breakdown.

1. To use the solution, customer can run the an AWS Systems Manager Runbook ( Automation Document ) called `TrustedAdvisor-WellArchitected-Optimization-Starter`, entering a few input parameters. Including the [AWS Well-Architected Tool](https://aws.amazon.com/well-architected-tool/) workload name to be use for the optimization review.

2. The AWS Systems Manager Runbook, will then  call an [AWS Lambda function](https://aws.amazon.com/pm/lambda/) asyncronously, and wait for the Lamba function to complete, using the `aws:approve`  [action](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-action-approve.html).

3. The Lambda function will first create, a temporary AWS Well-Architected Tool workload in `us-east-1` region, with the AWS Trusted Advisor discovery enabled. And gather all the mappings,  between Trusted Advisor Checks and Well-Architected Framework best practices. 

4. Once this mapping is populated, the Lambda function will then compile the list of AWS resources, detected by Trusted Advisor checks in the mapping.  And compile a HTML report, placed in an [Amazon S3](https://aws.amazon.com/pm/serv-s3/) bucket.

5. The URL to access the HTML report will placed in the note section of related Questions for best practices in the specified Well-Architected Tool workload in step 1. 

6. The HTML report will also be sent to the user via email. 



<details>
<summary>[ Click here for detailed diagram ]</summary>

![TA-WA-Optimizer-HighArch2.png](./static/images/TA-WA-Optimizer-HighArch2.png)

</details>


## Deploy Systems Manager Automation Document

1.  Open [AWS CloudShell](https://ap-southeast-2.console.aws.amazon.com/cloudshell/home?region=ap-southeast-2#) in ap-southeast-2 region.

2.  Run below commands to deploy the AWS Systems Manager Runbook and other supporting resources.

    ```
    git clone https://github.com/stephensalim/Trusted-Advisor-Tools.git
    cd Trusted-Advisor-Tools/TA-WellArchitected/tawa-optimization-starter/
    sam build 
    sam deploy --guided --resolve-s3 --capabilities CAPABILITY_NAMED_IAM
    ```

3.  Enter values of below parameters, and select default for the rest of the prompts

    - **Stack Name** = Enter the CloudFormation stack name for the app `tawa-opt`.
    - **AWS Region** = This is the AWS Region where the CloudFormation stack will be created `ap-southeast-2`.
    - **TAResourceReportBucket** = Enter a unique S3 Bucket for reporting `<accountid>-tawa-report` replace <accountid> with your account id. ( A new S3  bucket will be created, make sure the bucket is unique   )
    - **UseReportOwnURL** = Leave this field empty. 
    - **NotificationEmail** = This is the email address to notify users of the report url.
    
    
        <img src="./static/images/prompts.png" width="500" />
    
4. Check the Email Inbox of your specified email address in **NotificationEmail** parameter above, and click confirm subscription

    <img src="./static/images/subscribe.png" width="500" />

5.  Open [AWS Well-Architected Tool](https://ap-southeast-2.console.aws.amazon.com/wellarchitected/home?region=ap-southeast-2) in in ap-southeast-2 region
6.  o to Settings, and enable Discovery integration.

## Running Systems Manager Automation Document

1. Open AWS Systems Manager Console 
2. Click  **Documents** on the left menu area.
3. Then under  **Owned by me** tab , Click on the Automation Document called `TrustedAdvisor-WellArchitected-Optimization-Starter`.  

    <img src="./static/images/Shot01.png" width="500" />
    
4. Click `Execute Automation` and fill in the Input parameters. Particularly the `BestPracticeReviewName`.

    <img src="./static/images/Shot02.png" width="700" />
    
5. Wait until all Steps are completed ( this takes approx 10 minutes).
    
    <img src="./static/images/Shot04.png" width="700" />

6. Once all steps are completed successfully. 


7. Locate and run Review in the workload you specified, under the AWS Well-Architected Tool Console.

    <img src="./static/images/Shot05.png" width="500" />

8. Under the Notes section of Well-Architected questions, locate the URL to access the report. 

    <img src="./static/images/Shot06.png" width="500" />

9. Download the report

    <img src="./static/images/Shot07.png" width="500" />

10. Review the report

    <img src="./static/images/Shot08.png" width="700" />


# [ Optional ] - Deploy prioritization tool  

<img src="../tawa-eisenhower-matrix-app/static/images/aws_prioritization_matrix_app_sample.png" width="1000" />

In this section you will be deploying a react based prioritization tool, loosely inspired by **Eisenhower Matrix**. 
This tool takes input of the JSON file produced by the starter kit above, and allow you to visualize each improvement opportunities to assist with your prioritization.

**Eisenhower Matrix** is a productivity, prioritization, and time-management framework designed to help you prioritize a list of tasks or agenda items by first categorizing those items according to their urgency and importance.

<br>

> **Prerequisite:**
> Before proceeding with below steps, make sure you have [installed Node.js and npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)

<br>

<details>
<summary>[ Click here to expand instructions ]</summary>



### Host the Eisenhower Matrix App in AWS Amplify Hosting

1. Clone the repository, and navigate to the App directory from your local copy ``/Trusted-Advisor-Tools/TA-WellArchitected/tawa-eisenhower-matrix-app``.

    E.g. If your current directory is ``tawa-optimization-starter``, navigate to the right directory as below:

    ```
    cd ..
    cd tawa-eisenhower-matrix-app
    ```

2. Install dependencies and Build the application. After doing this, the ``/build`` directory will be created inside the current directory: 

   ```
    npm install
    npm run build
    ```

3. Navigate into the ``/build`` directory, and zip its content into a zipped file named ``build-eisenhower.zip``

    ```
    cd build
    zip -r build-eisenhower.zip .
    ```

4. Open the [AWS Amplify Console](https://us-east-1.console.aws.amazon.com/amplify/home?region=us-east-1#/) and click on **"Create new app"** for using Amplify Hosting:

    <img src="../tawa-eisenhower-matrix-app/static/images/aws_amplify_get_started_1.png" width="700" />

5. Select **Deploy without Git**:

    <img src="../tawa-eisenhower-matrix-app/static/images/aws_amplify_get_started_2.png" width="700" />

6. In the **Manual deploy** page, fill in the **App name** and **Branch name** fields. For **Method**, select **"Drag and drop"**. Drag and drop the ``/build/build-eisenhower.zip`` zipped file created before into the section, and click on **Save and deploy**:

    <img src="../tawa-eisenhower-matrix-app/static/images/aws_amplify_get_started_3.png" width="700" />

7. Once the deployment is completed, click on the Domain link generated by Amplify to open the Eisenhower Matrix App:

    <img src="../tawa-eisenhower-matrix-app/static/images/aws_amplify_get_started_4.png" width="700" />
    <img src="../tawa-eisenhower-matrix-app/static/images/aws_prioritization_matrix_app_sample_start.png" width="700" />

### Load the Trusted Advisor Check results into the Eisenhower Matrix App

Now that the Eisenhower Matrix App is running (either locally or via AWS Amplify), you need to load the Trusted Advisor Check results gathered from the **TA-WA Optimization** automation in previous steps.
<br><br>
> **Important Note:**
> This Eisenhower Matrix tool is a react front-end only application. This means that no data is stored in any backend server or component while using the app (all data remains client side).
<br>
1. Open the same S3 report link you opened during the **Step 8** under **Running Systems Manager Automation Document** section above.

2. Navigate to the **report/** directory of that S3 bucket:

    <img src="../tawa-eisenhower-matrix-app/static/images/s3_download_json_file_1.png" width="700" />

3. Open the **json/** folder:

    <img src="../tawa-eisenhower-matrix-app/static/images/s3_download_json_file_2.png" width="700" />

4. Select the **.json** report and click on **Download**:

    <img src="../tawa-eisenhower-matrix-app/static/images/s3_download_json_file_3.png" width="700" />

5. Open the Eisenhower Matrix App and click on **Upload json file**:

    <img src="../tawa-eisenhower-matrix-app/static/images/aws_prioritization_matrix_app_sample_upload.png" width="700" />

6. After loading the file into the application, the Trusted Advisor Checks table should open. Use this table to select the checks you would like to start prioritizing in the dashboard (Eisenhower Matrix):

    <img src="../tawa-eisenhower-matrix-app/static/images/aws_prioritization_matrix_app_sample_ta_checks_selection.png" width="700" />

7. Now, you can start prioritizing the checks based on the Business Impact and Urgency by moving the boxes to the different quadrants of the dashboard (Eisenhower Matrix):

    <img src="../tawa-eisenhower-matrix-app/static/images/aws_prioritization_matrix_app_sample_comments.png" width="700" />


## Cleanup - Deleting the Eisenhower Matrix App from AWS Amplify

1. In the AWS Amplify Console, open your application and click on **"Actions" > "Delete app"**:

    <img src="../tawa-eisenhower-matrix-app/static/images/aws_amplify_cleanup_1.png" width="500" />

2. In the confirmation window, type *"delete"* and click on the **"Delete"** button:

    <img src="../tawa-eisenhower-matrix-app/static/images/aws_amplify_cleanup_2.png" width="500" />
    
</details>


## Cleanup - TA-WA Optimization solution (SAM stack)

1. Delete the report S3 Bucket you specified **TAResourceReportBucket** during creation of the Stack.

2. Run below commands to delete the stack, and select yes for all prompts to delete resource.

    ```
    sam delete --stack-name {{ Stack Name }}
    ```
