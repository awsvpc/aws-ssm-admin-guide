Create automation document in EC2 Systems Manager (SSM):

aws ssm \
create-document \
--name "TVLK-ami-sans-bdm" \
--content "file://TVLK-ami-sans-bdm.json" \
--document-type "Automation"

When successful, you should see JSON output with document description, type, version and other meta data.

If you need to make a change, update your document by creating a new version with:

aws ssm \
update-document \
--name "TVLK-ami-sans-bdm" \
--content "file://TVLK-ami-sans-bdm.json" \
--document-version "\$LATEST"

Note: while I applaud AWS for incorporating versioning of documents and Lambdas directly into AWS, I would rather have better integration with popular VCS like Git.

List all documents you have created so far with:

aws ssm \
list-documents \
--document-filter-list "key=Owner,value=self"

To view specific document:

aws ssm \
get-document \
--name "TVLK-ami-sans-bdm"

In case you want to get your JSON back out of the document without meta data and escape characters, you can use this little jq trick:

aws ssm \
get-document \
--name "TVLK-ami-sans-bdm" \
--query 'Content' \
| jq '.|fromjson'

Now we can finally execute our Automation with the following command (just don't forget to substitute real values for AutomationAssumeRole, SubnetId and SourceAmiId):

aws ssm \
start-automation-execution \
--document-name "TVLK-ami-sans-bdm" \
--parameters \
"AutomationAssumeRole=arn:aws:iam::123456789012:role/YourAutomationRole,SubnetId=subnet-12345678,SourceAmiId=ami-XXXXXX"

If all is well, you should get back AutomationExecutionId as part of the output. Note this value, you will need it to poll status of your automation run from CLI with (again, substitute the real value for --automation-execution-id):

aws ssm \
get-automation-execution \
--automation-execution-id "12345678-1234-1234-1234-1234567890ab"

Alternatively, just use EC2 Console to monitor the execution.
