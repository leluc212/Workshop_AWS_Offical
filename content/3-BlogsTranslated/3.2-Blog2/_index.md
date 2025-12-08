---
title: "Blog 2"
date: "2025-09-08"
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---


**AWS DevOps & Developer Productivity Blog**  
# Exploring AWS Console: Diagnosing Errors with Amazon Q Developer
**Authors:** Marco Frattallone, Matthias Seeger, and Surabhi Tandon — January 10, 2025*  
**Category:** Amazon Q, Amazon Q Developer, AWS Management Console, DevOps, Intermediate (200)  
________________________________________  

## Introduction  
Developers, IT Operators, and sometimes Site Reliability Engineers (SREs) are responsible for deploying and operating infrastructure along with applications, and effectively troubleshooting issues promptly. Effective incident management requires fast diagnosis, root cause analysis, and corrective actions. Identifying root causes can be complex in modern systems, which include many resources deployed distributedly across various environments. Amazon Q Developer, an AI generative-assisted assistant, can simplify this process by diagnosing errors appearing in the AWS Management Console.  

Amazon Q Developer saves crucial time when handling production incidents by supporting diagnosis of errors related to your AWS environment. These errors may stem from misconfigurations across multiple resources, often requiring toggling between different AWS service consoles to find the root cause. Amazon Q Developer uses generative AI to automate the error diagnosis process within the AWS Console interface, reducing Mean Time To Repair (MTTR) and minimizing the impact on business operations.  

This article explains how Amazon Q Developer supports error diagnosis in the AWS Console when working with supported AWS services. It also describes how this feature works to guide you in troubleshooting. We analyze the behind-the-scenes processes that enable and support this feature.  

## Diagnosing with Amazon Q  
The Diagnose with Amazon Q feature helps diagnose most common errors occurring in the AWS Management Console for AWS services currently supported by this feature. It activates when a user with appropriate permissions clicks the Diagnose with Amazon Q button next to an error message. Amazon Q provides a natural language explanation analyzing the root cause of the error. Then, when the user clicks Help me resolve, Amazon Q displays an ordered list of step-by-step instructions to help you fix the issue. After completion, you can provide feedback on whether the solution provided by Amazon Q was helpful.  

An example illustrates how Amazon Q Developer can help diagnose an error when launching an Amazon EC2 instance and provide step-by-step guidance to resolve the error.  

### Diagnosing with Amazon Q – IAM Permissions related to EC2 instance launch error  

## Behind the Scenes: How Amazon Q Creates Diagnosis  
To illustrate the concepts, we present explanations in the context of two real examples.  

### Example 1:  
Suppose you try to delete an Amazon S3 bucket that is not empty. This leads to the error message:  
*This bucket is not empty. Buckets must be empty before they can be deleted. To delete all objects in the bucket, use the empty bucket configuration.*  

### Example 2:  
Suppose you try to list objects in a specific S3 bucket but lack AWS Identity and Access Management (IAM) permissions to perform the action. This leads to the error message:  
*Insufficient permissions to list objects. After you or your AWS administrator has updated your permissions to allow the s3:ListBucket action, refresh the page. Learn more about Identity and access management in Amazon S3.*  

When you click the Diagnose with Amazon Q button next to the error message in the AWS Management Console, Amazon Q generates an analysis explaining the root cause of the error in natural language. This step is supported by large language models (LLMs). Contextual information provided to the LLM includes the error message displayed in the console, the URL of the triggering action, and the IAM role of the user logged into the AWS Console. The service always operates within the permissions granted to your role when you interact with the AWS Console, and privileges never escalate beyond what you are assigned.  

When you click Help me resolve after reviewing the analysis, Amazon Q retrieves additional information about the state of resources in the AWS Account where the error occurred. At this stage, the system proactively determines what information is missing and generates interrogation requests to internal services to fulfill the informational needs. Interrogation is unnecessary for simple errors, such as Example 1 above, but becomes essential for resolving more complex errors when contextual information is insufficient.  

Based on the context, error analysis, user permissions, and account interrogation results, Amazon Q generates step-by-step Resolution instructions. This step is supported by LLMs.  

After deploying and validating the steps provided by Amazon Q to resolve the error in the console, you have the option to provide feedback on your experience.  

---

### Diagram illustrating interaction between User, AWS Console, and Amazon Q Developer  

## Contextual Information  
Contextual information helps large language models (LLMs) generate more accurate and relevant responses. Context data is automatically passed to Amazon Q from the AWS Console interface. As the foundation for the entire analysis and decision-making process, the context information should be provided as fully as possible. At minimum, Amazon Q collects the error message, the URL of the action that caused the error, and the IAM role of the logged-in user.  

The system automatically extracts relevant identifiers from this context information.  

In our running Example 1, the URL might be:  
`https://s3.console.aws.amazon.com/s3/bucket/my-bucket-123456/delete?region=us-west-2`  
from which Amazon Q extracts `aws_region = "us-west-2"` and `s3_bucket_name = "my-bucket-123456"`.  

Beyond this minimal context, Amazon Q may collect additional information from the console related to what the user sees on screen when the error occurs, such as content in text boxes or widgets in the current interface. Amazon Q can also use specific context provided by the underlying service platform. In Example 2, the bucket name is extracted from the URL, the action `s3:ListBucket` from the error message, and Amazon Q may fetch additional IAM information about relevant policies as well as allow/deny statements.  

## Interrogating the Logged-in User’s Account  
The Diagnose with Amazon Q feature not only passively receives context but can actively query for more information. Amazon Q performs read-only queries to gather additional context about resources in the AWS account, their states, and relationships with the resource experiencing the error. This relationship context helps the LLM model analyze the root cause more accurately when diagnosing errors.  

Amazon Q interrogates the logged-in user’s account using the AWS Cloud Control API (CCAPI) to identify resources deployed in the account. During initial onboarding of Amazon Q, the managed policy `AmazonQFullAccess` is attached to the IAM role the user employs. This policy includes the IAM permissions `cloudformation:ListResources` and `cloudformation:GetResource` for CCAPI, enabling access to the read and list endpoints of CCAPI. If you do not want to attach the managed `AmazonQFullAccess` policy, you can add `cloudformation:ListResources` and `cloudformation:GetResource` directly to your IAM Role.  

In simple Example 1, where the error occurs because the S3 bucket is not empty, the error message and console URL already contain sufficient information to address it, so no further queries to the AWS account are necessary. In contrast, for the IAM permissions error in Example 2, understanding the IAM role permissions attached to the affected resource is useful. Amazon Q can obtain information about identity-level policies for the role and resource-level policies for the affected resource. Based on that, Amazon Q analyzes the cause of the error by using internal IAM services.  

Specifically, in Example 2, the URL might be:  
`https://s3.console.aws.amazon.com/s3/buckets/my-bucket-123456?region=us-west-2&bucketType=general&tab=objects`  
from which Amazon Q extracts the region and bucket name. It can also extract the `s3:ListBucket` action directly from the error message. Based on this information, Amazon Q fetches bucket policies for `my-bucket-123456` and identity policies for the `ReadOnly` role, then scans to check whether the `s3:ListBucket` action is present or calls internal IAM services to collect more information about the cause of the access denial.  

Amazon Q operates only within the permission scope granted by the IAM role of the logged-in user, ensuring privileges never escalate beyond those assigned. Amazon Q calls CCAPI on behalf of the logged-in user, using exactly the permissions allowed by the user's IAM role. CCAPI inherits the user's permissions and has the same level of access to query resources in the user's account.  

In Example 2, if the logged-in user does not have access to the policies for bucket `my-bucket-123456`, Amazon Q will not be able to access them either. All API calls are logged in CloudTrail, including Amazon Q’s CCAPI calls, and CCAPI’s calls to target services (e.g., S3, IAM) depending on the request.  

## Creating Step-by-Step Resolution Instructions  
At this stage, all gathered information is synthesized by Amazon Q to create useful and actionable step-by-step remediation instructions. Below are sample instructions for the examples discussed. As models are updated and improved over time, feedback may evolve.  

For Example 1, sample instructions might be:  
1. Access the S3 console, click "Buckets" and select bucket `my-bucket-123456`.  
2. Click on the "Empty" tab.  
3. If the bucket contains a large number of objects, creating a lifecycle rule to delete all objects in the bucket might be a more efficient way to empty the bucket.  
4. Enter "permanently delete" into the text box and confirm that all objects will be deleted.  
5. Try deleting the S3 bucket `my-bucket-123456` again.  

For Example 2, you could:  
1. Go to the IAM console. Edit the IAM policy attached to the `ReadOnly` role.  
2. Allow the action `s3:ListBucket` for the resource with ARN: `arn:aws:s3:::my-bucket-123456`  
3. Save the updated IAM policy.  
4. Refresh the S3 console page to list objects in bucket `my-bucket-123456`.  

Note that instructions include inferred context such as bucket name `my-bucket-123456` rather than placeholders. The instructions returned by Diagnose with Amazon Q are complete and detailed enough for users to follow without additional steps.  

In practice, although the service uses LLMs to synthesize remediation instructions, Amazon Q applies post-processing to fix common mistakes. For example, in Example 2, the LLM might output the ARN as `arn:aws:s3:<region>::<bucket_name>`, and the system automatically corrects it as shown above.  

The returned instructions for Example 2 assume the user cannot list objects due to a missing Allow statement in the policies attached to the `ReadOnly` role. However, other root causes might be a Deny statement in a policy attached to the S3 bucket or the `ReadOnly` role. Diagnose with Amazon Q can use the account interrogation process to determine the exact root cause and propose appropriate solutions. For the example above, Amazon Q may fetch the policies attached to the `ReadOnly` role to check whether the `s3:ListBucket` permission is missing or retrieve bucket policies attached to `bucket-123456`.  

## Validation  
One key goal of Diagnose with Amazon Q is to maintain high quality standards, ensuring users always receive helpful and actionable recommendations whenever they encounter errors. An important prerequisite is a strong and flexible evaluation system. Evaluating generative AI systems is challenging due to the large output space (natural language) and non-deterministic responses.  

In summary, our validation system is built on a large dataset of errors, where each record has a certain number of annotations. Each record contains context (template error messages and Console URLs, e.g., `bucket-123456` replaced by `{{s3_bucket_name}}`, `us-west-2` replaced by `{{aws_region}}`). Annotations include descriptions of the Infrastructure as Code (CloudFormation) representing the account state with errors and the triggering action, as well as expert-provided correct responses. These records allow us to simulate system behavior variants without human interaction and run much faster than real-time by parallelization.  

We are also developing automated evaluation metrics to compare standard annotations with system responses, enabling fully automated offline evaluations.  

This validation system allows rapid testing of new ideas by comparing with the current state, preventing quality regressions. Although human experts are still needed to create annotations for error records, we continuously innovate to speed up and simplify this task by developing annotation tools designed to avoid natural language data entry, integrated with validation mechanisms, and requiring output editing rather than full re-annotation.  

## Conclusion  
The Diagnose with Amazon Q feature of Amazon Q Developer enables you to identify error root causes in the AWS Console without navigating multiple service consoles. By providing detailed, step-by-step instructions customized to your AWS account and specific error context, Amazon Q Developer helps you troubleshoot and resolve issues effectively. This helps your organization improve operational efficiency, reduce downtime, enhance service quality, and free up valuable human resources to focus on higher-value activities.  

We also provide insights into how AI and machine learning power this feature behind the scenes.  

## About the Authors  
**Matthias Seeger**  
Matthias Seeger is a Principal Applied Scientist at AWS. His interests include Bayesian learning and probabilistic model-based decision making, theory and applications of Gaussian Process models, probabilistic forecasting, and more recently, large language models (LLMs) and challenges in generating and labeling related data.  

**Marco Frattallone**  
Marco Frattallone is a Senior Technical Account Manager at AWS, focusing on partner support. He works closely with partners to build, deploy, and optimize solutions on AWS, providing guidance and sharing best practices. Marco is passionate about technology and helping partners lead innovation. Outside of work, he enjoys cycling, sailing, and exploring new cultures.  

**Surabhi Tandon**  
Surabhi Tandon is a Senior Technical Account Manager at Amazon Web Services (AWS). She supports enterprise customers to achieve high operational efficiency and assists them on their cloud journey with AWS by providing strategic technical guidance. Surabhi is passionate about Generative AI, automation, and DevOps. Outside of work, she enjoys hiking, reading, and spending time with family and friends.  
