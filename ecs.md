# Deploying Kimai on AWS ECS with RDS

This guide provides step-by-step instructions to deploy the **Kimai** time-tracking application on **AWS ECS (Fargate)** with **Amazon RDS (MySQL/PostgreSQL)** while following security best practices using **AWS Secrets Manager**.

## **1. Prerequisites**

Before starting, ensure you have:

- An **AWS account** with IAM permissions to manage ECS, RDS, and ALB.
- AWS CLI installed and configured with necessary permissions.
- An **Amazon RDS (MySQL/PostgreSQL) instance** created.
- An **Amazon ECS cluster** set up.
- A **VPC with private subnets** for RDS and ECS tasks.
- An **Application Load Balancer (ALB)** created.
- AWS **Secrets Manager** to securely store database credentials.

## **2. Store Sensitive Data in AWS Secrets Manager**

Before creating the ECS task definition, store the **database URL** and **trusted hosts** in **AWS Secrets Manager**.

### **2.1 Store Database Credentials**
Run the following command to store the `DATABASE_URL` securely:

```bash
aws secretsmanager create-secret --name kimai-db-credentials \
--secret-string '{"DATABASE_URL":"mysql://kimai_user:password@rds-hostname:3306/kimai_db"}'
```

### **2.2 Store Trusted Hosts**
Run the following command to store `TRUSTED_HOSTS` securely:

```bash
aws secretsmanager create-secret --name kimai-app-secrets \
--secret-string '{"TRUSTED_HOSTS":"^your-domain.com$"}'
```

## **3. Create an ECS Task Definition**

Save the following content as `kimai-task-def.json`:

```json
{
  "family": "kimai-task",
  "executionRoleArn": "arn:aws:iam::ACCOUNT_ID:role/ecsTaskExecutionRole",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [
    {
      "name": "kimai",
      "image": "ghcr.io/kimai/kimai2:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8001,
          "protocol": "tcp"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:region:ACCOUNT_ID:secret:kimai-db-credentials:DATABASE_URL"
        },
        {
          "name": "TRUSTED_HOSTS",
          "valueFrom": "arn:aws:secretsmanager:region:ACCOUNT_ID:secret:kimai-app-secrets:TRUSTED_HOSTS"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/kimai",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

### **3.1 Register the Task Definition**
Run the following command to register the task definition:

```bash
aws ecs register-task-definition --cli-input-json file://kimai-task-def.json
```

## **4. Create an ECS Service**

Save the following content as `kimai-service-def.json`:

```json
{
  "serviceName": "kimai-service",
  "cluster": "kimai-cluster",
  "taskDefinition": "kimai-task",
  "desiredCount": 2,
  "launchType": "FARGATE",
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": ["subnet-xxxxxxxx", "subnet-yyyyyyyy"],
      "securityGroups": ["sg-xxxxxxxx"],
      "assignPublicIp": "DISABLED"
    }
  },
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:region:ACCOUNT_ID:targetgroup/kimai-target-group/xxxxxxxxxxxx",
      "containerName": "kimai",
      "containerPort": 8001
    }
  ],
  "healthCheckGracePeriodSeconds": 60,
  "deploymentController": {
    "type": "ECS"
  }
}
```

### **4.1 Create the ECS Service**
Run the following command to create the ECS service:

```bash
aws ecs create-service --cli-input-json file://kimai-service-def.json
```

## **5. Configure AWS Load Balancer**
1. Ensure an **Application Load Balancer (ALB)** is set up with an HTTPS listener.
2. Attach the **Kimai ECS service** to the ALB **target group**.
3. Update security groups to allow traffic from the ALB.

## **6. Scaling and Monitoring**
- **Enable Auto Scaling** for ECS service:
  ```bash
  aws application-autoscaling register-scalable-target \
    --service-namespace ecs \
    --scalable-dimension ecs:service:DesiredCount \
    --resource-id service/kimai-cluster/kimai-service \
    --min-capacity 2 --max-capacity 5
  ```
- **Monitor logs in CloudWatch**:
  ```bash
  aws logs tail /ecs/kimai --follow
  ```

## **7. Security Best Practices**
- Store **database credentials** in **AWS Secrets Manager** instead of hardcoding them.
- Use **IAM roles** with **least privilege access** for ECS tasks.
- Ensure **RDS is in a private subnet** to restrict external access.
- Enable **automatic backups** for RDS to prevent data loss.
- Use **AWS WAF** to protect the ALB from malicious traffic.
- Enable **CloudTrail** for security auditing and monitoring.

## **8. Next Steps**
- Configure **Kimai admin user**.
- Set up **automated backups for RDS**.
- Enable **AWS GuardDuty** for enhanced security monitoring.

---
🚀 **Kimai is now running on AWS ECS securely using AWS Secrets Manager!** 🚀
