# Creating-AWS-Resources-with-Functions-and-Arrays


Creating AWS Resources with Functions and Arrays

Using shell scripting, you can automate the creation of AWS resources like EC2 instances and S3 buckets by leveraging functions and arrays for modularity and efficiency. The script starts by defining environment variables and helper functions to validate inputs, such as checking the number of arguments and ensuring the AWS CLI and profile are set. The create_ec2_instances function provisions EC2 instances with specified parameters (e.g., instance type t2.micro, AMI ID, and count), using the AWS CLI command aws ec2 run-instances, and includes error handling to confirm successful creation. Similarly, the create_s3_buckets function uses an array to store department names (e.g., Marketing, Sales) and loops through them to create uniquely named S3 buckets with a company prefix, employing the aws s3api create-bucket command. This approach ensures scalability and maintainability for managing cloud resources.

x

Copy
#!/bin/bash

# Environment variables
ENVIRONMENT=$1

# Function to check number of arguments
check_num_of_args() {
    if [ "$#" -ne 1 ]; then
        echo "Usage: $0 <environment>"
        exit 1
    fi
}

# Function to activate environment
activate_infra_environment() {
    if [ "$ENVIRONMENT" == "local" ]; then
        echo "Running script for Local Environment..."
    elif [ "$ENVIRONMENT" == "testing" ]; then
        echo "Running script for Testing Environment..."
    elif [ "$ENVIRONMENT" == "production" ]; then
        echo "Running script for Production Environment..."
    else
        echo "Invalid environment specified. Please use 'local', 'testing', or 'production'."
        exit 2
    fi
}

# Function to check if AWS CLI is installed
check_aws_cli() {
    if ! command -v aws &> /dev/null; then
        echo "AWS CLI is not installed. Please install it before proceeding."
        return 1
    fi
}

# Function to check if AWS profile is set
check_aws_profile() {
    if [ -z "$AWS_PROFILE" ]; then
        echo "AWS profile environment variable is not set."
        return 1
    fi
}

# Function to create EC2 instances
create_ec2_instances() {
    instance_type="t2.micro"
    ami_id="ami-0cd59ecaf368e5ccf"
    count=2
    region="eu-west-2"

    aws ec2 run-instances \
        --image-id "$ami_id" \
        --instance-type "$instance_type" \
        --count $count \
        --key-name MyKeyPair \
        --region "$region"

    if [ $? -eq 0 ]; then
        echo "EC2 instances created successfully."
    else
        echo "Failed to create EC2 instances."
    fi
}

# Function to create S3 buckets for departments
create_s3_buckets() {
    company="datawise"
    departments=("Marketing" "Sales" "HR" "Operations" "Media")

    for department in "${departments[@]}"; do
        bucket_name="${company}-${department}-Data-Bucket"
        aws s3api create-bucket --bucket "$bucket_name" --region eu-west-2
        if [ $? -eq 0 ]; then
            echo "S3 bucket '$bucket_name' created successfully."
        else
            echo "Failed to create S3 bucket '$bucket_name'."
        fi
    done
}

# Execute functions
check_num_of_args
activate_infra_environment
check_aws_cli
check_aws_profile
create_ec2_instances
create_s3_buckets
