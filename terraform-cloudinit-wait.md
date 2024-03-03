<pre>
  There are some prerequisites to use AWS SSM Run Command: we need to have AWS SSM Agent installed on our instance. It is preinstalled on Amazon Linux 2 and Ubuntu 16.04, 18.04, and 20.04. For any other OS, we need to install it manually: it is supported on Linux, macOS, and Windows.

📖
If you don’t know AWS SSM yet, go and take a look to their introductory guide.
The user or the role that executes the Terraform code needs to be able to create, update, and read AWS SSM Documents, and run SSM commands. A possible policy could be look like this:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1629387563127",
      "Action": [
        "ssm:CreateDocument",
        "ssm:DeleteDocument",
        "ssm:DescribeDocument",
        "ssm:DescribeDocumentParameters",
        "ssm:DescribeDocumentPermission",
        "ssm:GetDocument",
        "ssm:ListDocuments",
        "ssm:SendCommand",
        "ssm:UpdateDocument",
        "ssm:UpdateDocumentDefaultVersion",
        "ssm:UpdateDocumentMetadata"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
If we already know the name of the documents, or the instances where we want to run the commands, it is better to lock down the policy specifying the resources, accordingly to the principle of least privilege.

Last but not least, we need to have the AWS CLI installed on the system that will execute Terraform.

The Terraform code
After having set up the prerequisites as above, we need two different Terraform resources. The first will create the AWS SSM Document with the command we want to execute on the instance. The second one will execute such command while provisioning the EC2 instance.

The AWS SSM Document code will look like this:

resource "aws_ssm_document" "cloud_init_wait" {
  name = "cloud-init-wait"
  document_type = "Command"
  document_format = "YAML"
  content = <<-DOC
    schemaVersion: '2.2'
    description: Wait for cloud init to finish
    mainSteps:
    - action: aws:runShellScript
      name: StopOnLinux
      precondition:
        StringEquals:
        - platformType
        - Linux
      inputs:
        runCommand:
        - cloud-init status --wait
    DOC
}
We can refer such document from within our EC2 instance resource, with a local provisioner:

resource "aws_instance" "example" {
  ami           = "my-ami"
  instance_type = "t3.micro"
  provisioner "local-exec" {
    interpreter = ["/bin/bash", "-c"]
    command = <<-EOF
    set -Ee -o pipefail
    export AWS_DEFAULT_REGION=${data.aws_region.current.name}
    command_id=$(aws ssm send-command --document-name ${aws_ssm_document.cloud_init_wait.arn} --instance-ids ${self.id} --output text --query "Command.CommandId")
    if ! aws ssm wait command-executed --command-id $command_id --instance-id ${self.id}; then
      echo "Failed to start services on instance ${self.id}!";
      echo "stdout:";
      aws ssm get-command-invocation --command-id $command_id --instance-id ${self.id} --query StandardOutputContent;
      echo "stderr:";
      aws ssm get-command-invocation --command-id $command_id --instance-id ${self.id} --query StandardErrorContent;
      exit 1;
    fi;
    echo "Services started successfully on the new instance with id ${self.id}!"
    EOF
  }
}
From now on, Terraform will wait for cloud-init to complete before marking the instance ready.
      
</pre>
