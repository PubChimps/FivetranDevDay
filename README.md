# Fivetran Dev Day
| ![intro.png](images/intro.png) | 
|:--:| 

# Hands On Guide
This guide walksthrough how to use Fivetran, Redshift and Redshift ML to automate both the Data Ingestion and Machine Learning portions of an example data pipeline. Through the Fivetran AWS Dev Day, this guide shows how to get access to Event Engine, provision and set up Redshift, and create a Fivetran trial account through Redshift's partner integration.

| ![diagram.png](images/diagram.png) | 
|:--:| 

## Table of Contents
1. [Event Engine](#event)
2. [Redshift](#redshift)
3. [Fivetran](#fivetran)
4. [Redshift ML](#ml)

# Event Engine  <a name="event"></a>
The AWS Event Engine was created to help AWS field teams run Workshops, GameDays, Bootcamps, Immersion Days, and other events such as this Dev Day that require hands-on access to AWS accounts. This section instructs how to get access to Event Engine to create free, temporary AWS services. Start with the link provided during the hands on portion of Dev Day, *Accept Terms & Login*, then receive a One-Time Password via email to get access to Event Engine.


| ![event0.png](images/event0.png) | 
|:--:| 


| ![event1.jpg](images/event1.jpg) | 
|:--:| 
| *OTP via email is the preferred method for this Dev Day* |


| ![event2.jpg](images/event2.jpg) | 
|:--:| 
| *A password will be sent you the email specified to access free, temporary AWS services via Event Engine* |


| ![event3.jpg](images/event3.jpg) | 
|:--:| 
| *Enter the code sent from no-reply@us-east-1.otp.signin.aws.training* |


| ![event4.jpg](images/event4.jpg) | 
|:--:| 
| *This walkthrough will take place in the AWS Console and at Fivetran.com* |


| ![event5.jpg](images/event5.jpg) | 
|:--:| 



| ![event6.jpg](images/event6.jpg) | 
|:--:| 
| *Select Redshift in Recently visited services, or via search* |

# Redshift  <a name="redshift"></a>
This section is all about Redshift, and getting it ready for both Fivetran and Redshift ML. The pictures walkthrough how to create a cluster, configure IAM roles and S3 buckets so that Redshift ML can be used, and whitelist Fivetran IP addresses so that it can bring data into Redshift. This starts with creating a cluster

| ![rs1.jpg](images/rs1.jpg) | 
|:--:| 
| Select *Create cluster* |

| ![rs2.png](images/rs2.png) | 
|:--:| 
| Keep all options as their defaults, except **Admin user password** which is *Fivetran1* |

| ![rs18.jpg](images/rs18.jpg) | 
|:--:| 
| Find *S3* under *Services*, S3 is needed so that Redshift can store and reference Sagemaker assests. |

| ![rs19.jpg](images/rs19.jpg) | 
|:--:| 
| Select *Create bucket* |

| ![rs20.jpg](images/rs20.jpg) | 
|:--:| 
| When creating a bucket, its name must be globally unique, I achieved this by naming it redshiftml-*the email I used for this walkthrough*, then select *Create bucket* |

Once a Redshift Cluster and S3 bucket have been created, a specific IAM Role has to be create for Redshift to use AWS Sagemaker for ML purposes.

| ![rs3.jpg](images/rs3.jpg) | 
|:--:| 
| *IAM* can be found be searching the Serivces dropdown|

| ![rs4.jpg](images/rs4.jpg) | 
|:--:| 
| In *Roles* select *Create role*|

| ![rs5.png](images/rs5.png) | 
|:--:| 
| This will be a *Redshift - Customizable* role |

| ![rs6.jpg](images/rs6.jpg) | 
|:--:| 
| 2 policies will be needed for the role, add them by selecting *Create policy* |

| ![rs7.jpg](images/rs7.jpg) | 
|:--:| 
| Search for and add *AmazonS3FullAccess* |

| ![rs8.jpg](images/rs8.jpg) | 
|:--:| 
| Search for and add *AmazonSageMakerFullAccess* |

| ![rs9.jpg](images/rs9.jpg) | 
|:--:| 
| *No tags are needed* |

| ![rs10.jpg](images/rs10.jpg) | 
|:--:| 
| *redshiftml* is the **Role name**, and a description is needed before selecting *Create role* |

| ![rs11.jpg](images/rs11.jpg) | 
|:--:| 
| Once the role is created, its trust relationship must be edited, select *Edit trust relationship* (Note this is also where you can find the Role APN, which is needed later) |

| ![rs12.jpg](images/rs12.jpg) | 
|:--:| 
| After pasting the json below, select *Update Trust Policy* |

This trust policy allows Sagemaker to work on Redshift's behalf to automate the process of creating and training machine learning models automatically

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "redshift.amazonaws.com",
          "sagemaker.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

| ![rs13.jpg](images/rs13.jpg) | 
|:--:| 
| Back in Redshift, the newly created role will need to be applied to the cluster |

| ![rs14.jpg](images/rs14.jpg) | 
|:--:| 
| Select *Actions*, then *Manage IAM roles* |

| ![rs15.jpg](images/rs15.jpg) | 
|:--:| 
| Find the *redshiftml* role that was just created, then select *Associate IAM role* and *Save changes*|

| ![rs16.jpg](images/rs16.jpg) | 
|:--:| 
| Select *Actions* again, and *Modify publicly accessible setting* so that Fivetran can work with Redshift|

| ![rs17.jpg](images/rs17.jpg) | 
|:--:| 
| After ticking *Enable*, select *Save changes*|

Fivetran pulls data from sources and sends it to Redshift using a set of fixed IP addresses. To ensure that Fivetran can do this, some IP addresses must be whitelisted.

| ![rs21.jpg](images/rs21.jpg) | 
|:--:| 
| In the Redshift cluster's *Properties*, select the name of the *VPC security group* |

| ![rs22.jpg](images/rs22.jpg) | 
|:--:| 
| In the secruity group's *Inbound rules*, select *Edit Inbound rules* |

| ![rs23.jpg](images/rs23.jpg) | 
|:--:| 
| The two IP ranges below should be added as *Custom TCP* for *Port Range* 5439, the *Save Rules* |

`35.234.176.144/29`
`52.0.2.4/32`

| ![rs24.jpg](images/rs24.jpg) | 
|:--:| 
| Fivetran will be added to Redshift with *Add partner integration* |

# Fivetran <a name="fivetran"></a>
Here Fivetran is set up to automate the process of sending data to Redshift

| ![ft1.jpg](images/ft1.jpg) | 
|:--:| 
| Fivetran automates data ingestion from over 150 sources to destinations like Redshift |

| ![ft2.jpg](images/ft2.jpg) | 
|:--:| 
| Keep all them information as their default values and select *Add partner* |

| ![ft3.jpg](images/ft3.jpg) | 
|:--:| 
| To start a Fivetran trial, the **E-mail** and **Company** entered cannot have been used for a trial previously. To achieve this, I created an email for this Dev Day and used it as the Company. Select *Sign up* |

| ![ft4.jpg](images/ft4.jpg) | 
|:--:| 
| Select *Verify your Account* in an email from sales@fivetran.com |

| ![ft5.jpg](images/ft5.jpg) | 
|:--:| 
| Create a password and *Continue* |

| ![ft6.jpg](images/ft6.jpg) | 
|:--:| 
| A Fivetran trial account has been created! Just one connector will be used in this example, select *Set up a connector* |

| ![ft7.jpg](images/ft7.jpg) | 
|:--:| 
| Fivetran will move data from Google Sheets to Redshift, select *Google Sheets* and *CONTINUE SETUP*|

| ![ft8.jpg](images/ft8.jpg) | 
|:--:| 
| With the **Destination schema** as *google_sheets* and **Destination table** as *devday* Fivetran knows where in Redshift to store data, then select *Grant User Access* and *AUTHORIZE* to select a Google account the Fivetran can use to access the Google sheet. Any Google account will be able to. |

| ![ft9.jpg](images/ft9.jpg) | 
|:--:| 
| The **Sheet URL** is listed below, after copying it, select *alldata* for the **Named Range** and *SAVE & TEST*|

Google Sheet URL 
`https://docs.google.com/spreadsheets/d/1HbFO7anjm_luv_2xugZIvCtObWKfwChVSP12a2gFxLk/edit`

| ![ft10.jpg](images/ft10.jpg) | 
|:--:| 
| After all Connections tests have passed, select *CONTINUE* |

| ![ft11.jpg](images/ft11.jpg) | 
|:--:| 
| Now Redshift will be set up as a Fivetran Destination, select *Redshift* and *CONTINUE SETUP* |

| ![ft12.jpg](images/ft12.jpg) | 
|:--:| 
| Back in the AWS Console, copy the Redshift Cluster endpoint |

| ![ft13.jpg](images/ft13.jpg) | 
|:--:| 
| Paste the copied Endpoint as a **Host**, but delete :5394/dev portion from the end, those belong in **Port** and **Database**. With the **Password** as *Fivetran1* and *Connect directly* **Connection Method**, select *SAVE & TEST*|

| ![ft14.jpg](images/ft14.jpg) | 
|:--:| 
| After all Connections tests have passed, select *CONTINUE* |

| ![ft15.jpg](images/ft15.jpg) | 
|:--:| 
| Finish Fivetran setup by selecting *Start Inital Sync* |

# Redshift ML <a name="ml"></a>
This section demonstrates Redshift ML, and how it can be used to create and use machine learning models automatically and in SQL.

| ![ml1.jpg](images/ml1.jpg) | 
|:--:| 
| In the Redshift Query Editor select *Connect to database* |

| ![ml2.jpg](images/ml2.jpg) | 
|:--:| 
| *Connect* to *dev* as *awsuser* |

| ![ml3.jpg](images/ml3.jpg) | 
|:--:| 
| Using the IAM Role and S3 Bucket creating while setting up Redshift, run the query below |

```
CREATE MODEL redshiftml_model FROM (SELECT input_1,
        input_2,
        input_3,
        input_4,
        label
    FROM google_sheets.devday
    )
TARGET label FUNCTION ml_fn_redshiftml_auto
IAM_ROLE XXXXXX_YOUR_IAM_ROLE_XXXXXXX SETTINGS (
  S3_BUCKET XXXXXX_YOUR_S3_BUCKET_XXXXXXX
);
```
| ![ml4.jpg](images/ml4.jpg) | 
|:--:| 
| Redshift ML will begin to generate, train, and evaluate the best model for the data, it's status can be checked with the query below |

`select schema_name, model_name, model_state from stv_ml_model_info;`

| ![ml5.png](images/ml5.png) | 
|:--:| 
| When finished **model_state** will be *Model is Ready*. This will likely take longer than the time premitted for the Dev Day, up to an hour. |

| ![ml6.jpg](images/ml6.jpg) | 
|:--:| 
| Once the model is ready, use the query below to make a prediction with it, and follow its instuctions for the chance to win a prize! |

```
SELECT ml_fn_redshiftml_auto(
    input_1,
    input_2,
    input_3,
    input_4)
    AS active FROM google_sheets.devday limit 1;
```
