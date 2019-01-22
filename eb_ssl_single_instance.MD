### How to install ssl certificate on aws elastic-beanstalk single instance environment?

1. create "mysite.crt" and "mysite.key" as required in nginx. 

1. create a s3 bucket "mys3bucket" and make it private. 

1. Update the bucket permission

    ```js
    {
        "Version": "2008-10-17",
        "Statement": [
            {
                "Sid": "<random_id>",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::<account_id>:role/aws-elasticbeanstalk-ec2-role"
                },
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::mybucket/*"
            }
        ]
    }
    ```

1. create `.ebextensions/files.config` file at root of beanstalk project. Add the following code block. This block of code will download the certificates from s3.
    ```yml
    files:
     /etc/nignx/mysite.crt:
        source: https://s3-<region>.amazonaws.com/mybucket/mysite.crt
        authentication: S3Access
     /etc/nginx/mysite.key:
        source: https://s3-<region>.amazonaws.com/mybucket/mysite.key
        authentication: S3Access

    Resources:
      AWSEBAutoScalingGroup:
        Metadata:
          AWS::CloudFormation::Authentication:
            S3Access:
              type: S3
              roleName: aws-elasticbeanstalk-ec2-role
              buckets: mybucket
    ```

1. create `.ebextensions/proxy.config` file at root of beanstalk project. Add the following code block. This block of code will add required nginx configuration for ssl.
    ```yml
    files:
      "/etc/nginx/conf.d/proxy.conf":
        mode: "000400"
        owner: root
        group: root
        encoding: plain
        content: |
            map $http_upgrade $connection_upgrade {
                default        "upgrade";
                ""            "";
            }

            server {
                listen 443 default ssl;
                server_name  *.mysite.com;

                ssl_certificate    /etc/nginx/mysite.crt;
                ssl_certificate_key  /etc/nginx/mysite.key;

                location / {
                    proxy_pass     http://nodejs;
                    proxy_set_header        Connection "";
                    proxy_http_version 1.1;
                    proxy_set_header        Host            $host;
                    proxy_set_header        X-Real-IP       $remote_addr;
                    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
                }

            }
    ```
1. last but not least: allow traffic from port 443
1. done.
