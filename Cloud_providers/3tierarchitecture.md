# What is Three-Tier Architecture?

Three-tier architecture is a client-server software architecture pattern that divides an application into three distinct layers: the presentation layer, the logic layer, and the data layer. This separation of concerns allows for more modular, scalable, and maintainable applications.

## Presentation Layer
This is the topmost layer, the one users interact with directly. Think of it as the front end of your application. It handles the user interface and user experience, processing user input and displaying the output. Common technologies used here include HTML, CSS, JavaScript, and front-end frameworks like React or Angular.

## Logic Layer
The middle layer is where the magic happens. It contains the business logic, the set of rules and operations that define how data can be created, displayed, stored, and changed. This layer acts as a mediator between the presentation and data layers, processing inputs from the presentation layer and making calls to the data layer. Technologies often used here include server-side languages and frameworks like Java, .NET, Python with Django, or Node.js.

## Data Layer
At the bottom, we have the data layer. This layer is responsible for storing and retrieving data. It includes database servers and data access layers that manage CRUD (Create, Read, Update, Delete) operations. Relational databases like MySQL, PostgreSQL, and NoSQL databases like MongoDB are commonly used here.

# Setting up a Three-Tier Architecture on AWS

The first step is to set up a VPC for your application. The VPC will consist of 2 public subnets and 2 private subnets, these numbers can change as per requirement and complexity of your application. You can attach a NAT gateway to your private subnet if you want your private subnet to be able to download packages.

Then we need to decide where to put the presentation layer, it can be deployed on Amazon EC2 or S3. We use Amazon S3 to store static parts of your application like the frontend and images. In most of the cases the frontend is the static part of your application and it's best to keep it on Amazon S3.

1. **Create an S3 Bucket**:
   - Go to the S3 dashboard and create a new bucket.
   - Upload your frontend code there. If you have to store images for your application you can create a frontend and images folder separately or have separate buckets altogether for it.

2. **Set Up CloudFront**:
   - Go to CloudFront and create a new distribution.
   - Set the origin domain to the S3 bucket’s website endpoint.
   - Create a new origin and attach your backend application ALB there.
   - Create separate behaviors for your frontend and backend like caching and TTL.
   - Set the default root object to index.html.
   - Add your domain and certificate.

Now comes the Logic Layer or the backend service. Amazon EC2 is the best place to store your backend application. The backend server does not need an internet connection, so we should put it into a private subnet. We can also use AWS ECS for this use case.

1. **Launch EC2 Instances**:
   - Go to the EC2 dashboard and launch an instance.
   - Choose an AMI (Amazon Machine Image), like Amazon Linux 2 or Ubuntu.
   - Select the instance type (e.g., t2.micro for small applications).
   - Place the instance in the private subnet.

2. **Configure Security Groups**:
   - Create a security group allowing HTTP (port 80) and HTTPS (port 443) traffic.
   - Attach this security group to the EC2 instance.

3. **Set Up a Load Balancer**:
   - Set up a load balancer to handle SSL and access your backend service.
   - Change the security group of the backend server to only allow traffic from the ALB security group.
   - ALB is also important in case you need to set up scaling for your backend service.

Then comes the Data Layer. You can set up Amazon EC2 to host your database service or you can use a fully managed AWS service called RDS.

1. **Create an RDS Instance**:
   - Go to the RDS dashboard and create a new database instance.
   - Choose the database engine (e.g., MySQL, PostgreSQL).
   - Place the instance in the private subnet.

2. **Configure Security Groups**:
   - Create a security group that only allows traffic from the logic layer.
   - Attach this security group to the RDS instance.

The last part is the CDN part to make your application available globally and it also handles the security part of your application.

1. **Set Up CloudFront**:
   - Go to CloudFront and create a new distribution.
   - Set the origin domain to the S3 bucket’s website endpoint.
   - Create a new origin and attach your backend application ALB there.
   - Create separate behaviors for your frontend and backend like caching and TTL.
   - Set the default root object to index.html.
   - Add your domain and certificate.

2. **Configure DNS**:
   - Go to where you store your DNS records and add the CloudFront URL.

# Conclusion

Setting up a three-tier application on AWS using S3 and CloudFront for the frontend, along with EC2 and RDS for the backend, involves several steps, but the benefits of scalability, security, and maintainability are well worth the effort. By following this guide, you can create a robust and efficient architecture for your applications. Whether you’re building a simple web app or a complex enterprise system, leveraging AWS's powerful services will help you achieve your goals.


