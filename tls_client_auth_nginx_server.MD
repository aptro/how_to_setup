### How to build web server to do tls client authentication?

Stack
- Nginx
- Openssl
- Node/Express/Sails

Design
- ```openssl``` cmd tool chain for generating CA, private key and client certs
- ```nginx``` for authenticating the client and passing the information to the web server. 
- Most of the web server supports for tls authentication, but nginx implementation is way simpler and faster than doing in we server.


1. Create the Cirtificate Authority(CA)
    ```
    openssl req -new -x509 -days 9999 -keyout ca-key.pem -out ca-crt.pem
    ```
    Once you run the command it will ask for bunch of inputs to fill.


1. Generate a client private key
    ```
    openssl genrsa -out client-key.pem 4096
    ```

1. Generate the cirtificate signing request
    ```
    openssl req -new  -key client-key.pem -out client-csr.pem
    ```
Once you run the command it will ask for bunch of inputs to fill.

1. Sign and generate the client certs with the CA we generated earlier
    ```
    openssl x509 -req -days 999 -passin "pass:password" -in client-csr.pem -CA ca-crt.pem -CAkey ca-key.pem -CAcreateserial -out client-crt.pem
    ```

1. Verfiy the newly created crts
    ```
    openssl verify -CAfile ca-crt.pem client-crt.pem
    ```

1. Convert ca-crt.pem to nginx readable format
    ```
    openssl x509 -outform der -in ca-crt.pem -out ca.crt
    ```

1. Nginx configuration to validate the client
    ```
    server {
                listen 443 default ssl;
                server_name  *.server.com;

                ssl_client_certificate path_to/ca.crt;
                ssl_verify_client optional;

                location / {
                    proxy_pass     http://nodejs;
                    proxy_set_header        Connection "";
                    proxy_http_version 1.1;
                    proxy_set_header        Host            $host;
                    proxy_set_header        X-Real-IP       $remote_addr;
                    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header        SSL-CLIENT-VERIFIED $ssl_client_verify;
                    proxy_set_header        SSL-CLIENT-DN $ssl_client_s_dn;
                }

            } 
    ```

1. In nginx we are authenticating the client and passing the verification result the web server for Authorization, this give the web server to run additional logic example get user from database and verifyging the permissions.

1. Nodejs middleware sample code for client authoriztion
    ```
    function (req, res, next) {
      // nginx returns SUCCESS if it able to verify with the ca
			if (req.headers["ssl-client-verified"] != "SUCCESS") {
				return next();
			};
    
			if (typeof req.headers["ssl-client-dn"] != 'string') {
				return next();
			}

			var dn = req.headers["ssl-client-dn"].split(",");
			var dn_json = {}
			_.forEach(dn, function (each) {
				try {
					var each_arr = each.split('=')
					dn_json[each_arr[0]] = each_arr[1]
				} catch (error) {
					sails.info.error(error);
				};
			});
			// no common name found, return next
			if (!dn_json.CN)
				return next();
      // find a user from CN, here CN is the user id 
			User.findOne({ id: dn_json.CN }).exec(function (err, s) {
				if (err)
					return next(err);
				if (!u)
					return next();
				req.user = u
				return next()
			});
		}
    ```
  
1. Verifying the server setup works. We can create a simple node client to make a request to the server.
    ```
    var fs = require('fs');
    var https = require('https');
    var options = {
      hostname: 'server.com',
      port: 443,
      path: '/test',
      method: 'GET',
      key: fs.readFileSync('client-key.pem'),
      cert: fs.readFileSync('client-crt.pem')
    };
    var req = https.request(options, function (res) {
      res.on('data', function (data) {
        process.stdout.write(data);
      });
    });
    req.end();
    req.on('error', function (e) {
      console.error(e);
    });
    ```


### Note:

```ssl_verify_client``` options: on | off | optional | optional_no_ca;
The optional parameter requests the client certificate and verifies it if the certificate is present.

The optional_no_ca parameter requests the client certificate but does not require it to be signed by a trusted CA certificate.


### Ref:
HTTPS Authorized Certs with Node.js: https://engineering.circle.com/https-authorized-certs-with-node-js-315e548354a2

Cloudflare TLS client authentication: https://blog.cloudflare.com/introducing-tls-client-auth/

