AWSTemplateFormatVersion: '2010-09-09'
Description: >
  SPA hosting with secured API using Cognito authentication

Parameters: {}

Resources:
  # Cognito User Pool
  CognitoUserPool:
    Type: AWS::Cognito::UserPool
    Properties:
      UserPoolName: SampleUserPool
      AutoVerifiedAttributes:
        - email
      Policies:
        PasswordPolicy:
          MinimumLength: 8
          RequireUppercase: true
          RequireLowercase: true
          RequireNumbers: true
          RequireSymbols: false

  # Cognito User Pool Client
  CognitoUserPoolClient:
    Type: AWS::Cognito::UserPoolClient
    Properties:
      ClientName: SampleUserPoolClient
      UserPoolId: !Ref CognitoUserPool
      GenerateSecret: false
      AllowedOAuthFlows:
        - code
        - implicit
      AllowedOAuthScopes:
        - email
        - openid
        - profile
      CallbackURLs:
        - https://example.com/callback
      LogoutURLs:
        - https://example.com/logout
      SupportedIdentityProviders:
        - COGNITO
      AllowedOAuthFlowsUserPoolClient: true

  # Simple REST API
  SimpleAPI:
    Type: 'AWS::ApiGateway::RestApi'
    Properties:
      Description: A simple REST API
      Name: SimpleAPI
      EndpointConfiguration:
        Types:
          - REGIONAL

  # API Gateway Cognito Authorizer
  CognitoAuthorizer:
    Type: AWS::ApiGateway::Authorizer
    Properties:
      Name: CognitoAuthorizer
      IdentitySource: method.request.header.Authorization
      RestApiId: !Ref SimpleAPI
      Type: COGNITO_USER_POOLS
      ProviderARNs:
        - !GetAtt CognitoUserPool.Arn

  # /hello Resource
  SimpleAPIResource:
    Type: 'AWS::ApiGateway::Resource'
    Properties:
      ParentId: !GetAtt 
        - SimpleAPI
        - RootResourceId
      PathPart: hello
      RestApiId: !Ref SimpleAPI

  # GET Method with Cognito Auth
  HelloAPIGETMethod:
    Type: 'AWS::ApiGateway::Method'
    DependsOn:
      - CFDistributionSPA
    Properties:
      AuthorizationType: COGNITO_USER_POOLS
      AuthorizerId: !Ref CognitoAuthorizer
      HttpMethod: GET
      Integration:
        Type: MOCK
        PassthroughBehavior: WHEN_NO_MATCH
        RequestTemplates:
          application/json: "{\n \"statusCode\": 200\n}"
        IntegrationResponses:
          - StatusCode: 200
            SelectionPattern: 200
            ResponseParameters:
              method.response.header.Access-Control-Allow-Origin: "'*'"
            ResponseTemplates:
              application/json: "{\"message\": \"Hello World!\"}"
      MethodResponses:
        - StatusCode: 200
          ResponseParameters:
            method.response.header.Access-Control-Allow-Origin: true
          ResponseModels:
            application/json: Empty
      RestApiId: !Ref SimpleAPI
      ResourceId: !Ref SimpleAPIResource

  # API Deployment
  Deployment:
    Type: 'AWS::ApiGateway::Deployment'
    DependsOn:
      - HelloAPIGETMethod
    Properties:
      RestApiId: !Ref SimpleAPI
      StageName: v1

  # Remaining S3/CloudFront/Logging bucket definitions are omitted here
  # but assumed to be integrated as in your original template.

Outputs:
  UserPoolId:
    Value: !Ref CognitoUserPool
  UserPoolClientId:
    Value: !Ref CognitoUserPoolClient
  APIURL:
    Value: !Sub "https://${SimpleAPI}.execute-api.${AWS::Region}.amazonaws.com/v1/hello"
    Description: API endpoint protected by Cognito
