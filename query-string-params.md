# Parsing query string parameters inside a lambda function
When you want to handle calls to the API gateway that contain query sring params like so
```
https://www.example.com/query?filterData=' + encodeURI(JSON.stringify(filterData))
```
You must make sure to provide the api gateway with a template which will pass the data through to the Lambda function
### The Key Line:
```application/json: '{ "filterData" : $util.urlDecode($input.params().querystring.filterData) }'```
Once you do that the query string data will be accessable inside the lambda handler at _event.filterData_ or whatever query string
param key you choose 
##### Note: You don't have to json.parse(event.filterData)


## Example function definition
```    
queryReports:
    warmup: true
    handler: src/queryReports/handler.queryReports
    vpc:
      securityGroupIds:
        - sg-92ea14ec
      subnetIds:
        - subnet-44b51b3d
        - subnet-2118917b
        - subnet-0733fa4c
    package:
      include:
        - src/queryReports/**
        - node_modules/**
    events:
      - http:
          path: /reports/query
          method: get
          cors: true
          integration: lambda
          request:
            template:
              application/json: '{ "filterData" : $util.urlDecode($input.params().querystring.filterData) }'
          authorizer:
            arn: arn:aws:cognito-idp:us-west-2:445238294384:userpool/${file(config/config.${opt:stage, self:custom.default_stage}.json):user_pool_id}
    role: arn:aws:iam::445238294384:role/lambda_execution_role
    environment:
      DBHOST: ${file(config/config.${opt:stage, self:custom.default_stage}.json):db_host}
      DBNAME: ${file(config/config.${opt:stage, self:custom.default_stage}.json):db_name}
      DBUSER: ${file(config/config.${opt:stage, self:custom.default_stage}.json):db_user}
      DBPASSWORD: ${file(config/config.${opt:stage, self:custom.default_stage}.json):db_password}
```