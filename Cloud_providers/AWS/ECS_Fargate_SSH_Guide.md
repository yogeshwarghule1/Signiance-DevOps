
# Detailed Guide to SSH into AWS ECS Fargate Server and Container

## Prerequisites:
- Ensure the required policy is added to the Task Role attached to the Task Definition containing your container.
- Verify AWS CLI and AWS Session Manager Plugin are installed on your local machine.

## Steps to SSH into the Container:

### 1. Enabling ECS Exec for Tasks and Services:
You can enable the ECS Exec feature for your tasks and containers using the `--enable-execute-command` flag with the AWS CLI commands `create-service`, `update-service`, `start-task`, or `run-task`.

Use the following CLI command to enable ECS Exec for the respective task:
```bash
aws ecs update-service --cluster cluster-name --task-definition task-definition-name --enable-execute-command --service service-name --desired-count 1
```
**Note:** Before executing the above command, ensure your local PC is configured with the appropriate AWS role using the `aws configure` command and inputting the Access Key and Secret Access Key created while setting up the IAM user.

### 2. Verify ECS Exec Enablement:
After enabling ECS Exec, verify it with the following command:
```bash
aws ecs describe-tasks --cluster cluster-name --tasks task-id
```
If the output shows `enableExecuteCommand = False`, you may need to stop and restart the running task to reflect the changes. Ensure you update the `task-id` if it changes due to the restart.

### 3. SSH into the Container:
Once `enableExecuteCommand` is `True`, SSH into the Fargate server using the following command:
```bash
aws ecs execute-command --cluster cluster-name --task task-id --container container-name --interactive --command "/bin/sh"
```
If successful, you will be logged into the server and can execute commands within the container.

## Example Commands:
```bash
# Enable ECS Exec
aws ecs update-service --cluster my-cluster --task-definition my-task-def --enable-execute-command --service my-service --desired-count 1

# Verify ECS Exec Enablement
aws ecs describe-tasks --cluster my-cluster --tasks my-task-id

# SSH into the Container
aws ecs execute-command --cluster my-cluster --task my-task-id --container my-container --interactive --command "/bin/sh"
```

## Important Notes:
- Ensure IAM roles and policies are correctly set up for ECS Exec.
- Use the AWS CLI `aws configure` command to set up your local machine for accessing AWS resources.
- Restart the task if necessary to apply the `enableExecuteCommand` changes.

By following these steps, you can securely SSH into your AWS ECS Fargate container and perform necessary administrative tasks or troubleshooting.
