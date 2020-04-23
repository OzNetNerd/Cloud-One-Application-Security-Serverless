# Cloud One Application Security - Severless demo

A demonstration of how Trend Micro's "Cloud One: Application Security" product can be used to protect serverless applications.

More information can be found [here](https://www.trendmicro.com/en_au/business/products/hybrid-cloud/cloud-one-workload-security.html), as well as in the [docs.](https://cloudone.trendmicro.com/docs/application-security/introduction/#)

## Deployment
### Lambda layer

1. Create an Application Security Lambda Layer:
    
   ```
    mkdir /tmp/python
    cd /tmp/python
    pip3 install --target=/tmp/python trend-app-protect
    zip -r trend.zip /tmp/python
    aws lambda publish-layer-version \
        --layer-name app-protect \
        --description "App Protect" \
        --zip-file fileb://trend.zip \
        --compatible-runtimes python3.7
    ```

### Deploy app
 
1. Obtain the Layer ARN.
2. Create the CloudFormation stack:
 
     ```
    aws cloudformation create-stack \
    --stack-name ApplicationSecurityDemo \
    --template-body file://cfn.yaml \
    --parameters ParameterKey=ApplicationSecurityLayerArn,ParameterValue=<LAYER_ARN> \
    --capabilities CAPABILITY_IAM
    ```

## Attack the unprotected app

1. Obtain the app's URL:

    ```
    aws cloudformation describe-stacks --stack-name ApplicationSecurityDemo --query "Stacks[0].Outputs[?OutputKey=='AppUrl'].OutputValue" --output text    
    ```

2. Enter the following string into the form:

    ```
   ../../proc/self/environ
    ```
   
Among other things, the following values are shown:

* AWS_ACCESS_KEY_ID
* AWS_SECRET_ACCESS_KEY
* AWS_SESSION_TOKEN

These keys can be used by an attacker to access and control your AWS account.

## Protecting the app

1. Log into [Application Security.](https://dashboard.app-protect.trendmicro.com/)
2. Create a new "Group".
3. Set "Illegal File Access" to "**Mitigate**". 
4. Obtain the `Key` and `Secret` credentials.
5. Use these credentials to set the Lambda's `TREND_AP_KEY` and `TREND_AP_SECRET` API keys.
6. Try exploiting the app again:

    ```
   ../../proc/self/environ
    ```
   
# Contact

* Blog: [oznetnerd.com](https://oznetnerd.com)
* Email: [will@oznetnerd.com](mailto:will@oznetnerd.com)
