# AWS S3 CloudTrail Lake

Trying out AWS Cloudtrail Lake for querying S3 object events

## Set some variables

```
export account_id="xxxxx"
export profile="default"
export region="eu-west-1"
```

## Create Trail

```
aws cloudtrail create-trail \
    --name s3-object-trail \
    --s3-bucket-name my-cloudtrail-lake-bucket \
    --is-multi-region-trail \
    --region $region \
    --profile $profile
```

## Enable data events

```
aws cloudtrail put-event-selectors \
    --trail-name s3-object-trail \
    --advanced-event-selectors '[{
        "Name": "Log all S3 events for specific buckets",
        "FieldSelectors": [
            {"Field": "eventCategory", "Equals": ["Data"]},
            {"Field": "resources.type", "Equals": ["AWS::S3::Object"]},
            {"Field": "resources.ARN", "Equals": ["arn:aws:s3:::my-cloudtrail-lake-bucket/*", "arn:aws:s3:::my-test-bucket-gordon/*"]}
        ]
    }]' \
    --region $region \
    --profile $profile
```

## Verify Trail

```
aws cloudtrail get-event-selectors \
    --trail-name s3-object-trail \
    --region $region \
    --profile $profile
```

## Start Logging

```
aws cloudtrail start-logging \
    --name s3-object-trail \
    --region $region \
    --profile $profile
```

## Push a file to s3

```
aws s3 cp image.jpg s3://my-cloudtrail-lake-bucket/ \
    --profile $profile \
    --region $region
```

## Wait for Logs to Appear

CloudTrail typically delivers logs within 15 minutes of the API call, but it can sometimes take longer. Wait a few minutes before proceeding to the next step.

## Create a CloudTrail Lake Event Data Store

```
aws cloudtrail create-event-data-store \
    --name s3-object-datastore \
    --region $region \
    --profile $profile
```


## Query CloudTrail Lake Using SQL

```
EventDataStoreId="xxxxxx"

aws cloudtrail query \
    --query-statement "SELECT eventSource, eventName, eventTime, requestParameters FROM $EventDataStoreId WHERE eventName IN ('PutObject', 'DeleteObject')" \
    --region $region \
    --profile $profile
```