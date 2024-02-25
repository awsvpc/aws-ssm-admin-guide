<pre>
  Bash script which uses aws ssm send-command with --output-s3-bucket-name parameter to run the command and the result is stored in the S3 bucket, then displayed to the standard output.

#/usr/bin/env bash -xe
# Script to run PowerShell script on the Windows instance, then uploads the output to S3 bucket.
instanceId="$1"
bucketName="$2"
bucketDir="Output"
[ $# -le 2 ] && { echo "Usage: $0 instance_id bucket_name command"; exit 1; }
aws s3 ls ${bucketName} > /dev/null
cmdId=$(aws ssm send-command --instance-ids "$instanceId" --document-name "AWS-RunPowerShellScript" --query "Command.CommandId" --output text  --output-s3-bucket-name "$bucketName" --output-s3-key-prefix "$bucketDir" --parameters commands="'${@:3}'")
while [ "$(aws ssm list-command-invocations --command-id "$cmdId" --query "CommandInvocations[].Status" --output text)" == "InProgress" ]; do sleep 1; done
outputPath=$(aws ssm list-command-invocations --command-id "$cmdId" --details --query "CommandInvocations[].CommandPlugins[].OutputS3KeyPrefix" --output text)
aws s3 ls s3://${bucketName}/${outputPath}/stderr.txt && aws s3 cp --quiet s3://${bucketName}/${outputPath}/stderr.txt /dev/stderr
aws s3 cp --quiet s3://${bucketName}/${outputPath}/stdout.txt /dev/stdout
Example:

./run_ec2_ps_cmd_s3.sh i-0xyz my-bucket-name 'While ($i -le 10) {(Invoke-WebRequest -UseBasicParsing -Uri http://example.com).Content; $i++}'
</pre>
