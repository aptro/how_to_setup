### Publishing api documentation using Slate generator

#### Stack
* Gitlab-ci for building the assets and pushing into s3
* S3
* Lambda for Authorizer
* Api gateway
    * for Auth integration
    * S3 integration for serving static files
    * cname mapping in gateway


#### gitlab-ci.yml config
```
deploy_to_s3:
 type: deploy
 image: ruby
 only: 
  - master
 script:
  - apt update
  - apt install -yq python3-pip nodejs git
  - pip3 install awscli
  - bundle config build.nokogiri --use-system-libraries
  - bundle install
  - bundle exec middleman build
  - aws s3 sync build/ s3://my_static_site --exclude ".DS_Store/*" --cache-control "max-age=120000" --delete

```




Details about how to build the stack
https://docs.aws.amazon.com/apigateway/latest/developerguide/integrating-api-with-aws-services-s3.html
