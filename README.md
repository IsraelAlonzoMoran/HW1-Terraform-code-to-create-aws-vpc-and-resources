# HW1-Terraform-code-to-create-aws-vpc-and-resources
Terraform homework1, terraform code to create an aws vpc, internet_gateway, 3 public subnets, 3 private subnets, 2 RouteTables (1 public and 1 private), Elastic IP, NAT Gateway and the respective associations between the resources.

Step 1: Download the Terraform  package for the OS(this terraform code was tested in Ubuntu-20.04.4-amd64 64-bit) in the following link: https://www.terraform.io/downloads

Step 2: Unzip the downloaded terraform package(The terraform package will look like "terraform_1.1.9_linux_amd64.zip", unzip it).

Step 3: Once the package is unzipped "terraform_1.1.9_linux_amd64", open it and copy the binary called terraform, then paste the binary called "terraform" in the host machine path /usr/local/bin.

Step 4: Open the host machine Linux terminal and type $terraform --version (it will show the terraform version).

Step 5: Open the host machine Linux terminal and type $aws configure(enter the AWS_ACCESS_KEY_ID, then the AWS_SECRET_ACCESS_KEY, hit enter if the region is correct otherwise type it(us-west-2), then hit enter again for the output format(these steps will help to test that access to the AWS account is allow, cause to create an aws vpc and the rest of the vpc resources access to the aws account with the right permissions to create resources is required).

Step 6:$ git clone https://github.com/IsraelAlonzoMoran/HW1-Terraform-code-to-create-aws-vpc-and-resources.git

Step 7: Once the "provider.tf" code is downloaded to the host machine, with $cd go to the directory that has the downloaded folder, example "$cd /home/alonzo/Downloads/HW1-Terraform-code-to-create-aws-vpc-and-resources".

Step 8: Once in the directory that has the "provider.tf" file, type $terraform init, then type $terraform apply.

Step 9: Go to the AWS Console(https://aws.amazon.com/) select Sing in, enter your credentinals, type vpc in the search option and review that the aws vpc and rest of resources have been created.

That's all. Thank you.
