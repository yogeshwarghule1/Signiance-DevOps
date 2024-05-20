# Saving Costs on Your Cloud Storage: Moving Specific Content to Glacier Deep Archive

Optimizing storage costs is a crucial aspect of managing data effectively in today's cloud-centric environment. Amazon S3 provides a powerful platform for storing vast amounts of data, but as data accumulates, so do storage costs. In this blog post, we'll delve into a scenario where a customer faces the challenge of reducing S3 costs for a large dataset while maintaining accessibility to certain files. We'll explore how a tailored Python script provided an efficient solution to this problem.

## The Challenge

Our customer possessed an Amazon S3 bucket housing approximately 7TB of data, consisting of folders and MP4 files. The client's primary concern was the escalating cost associated with maintaining all objects in standard storage. They aimed to transition the contents of folders, along with any files starting with the prefix "RT," to Glacier Deep Archive for cost savings, while retaining MP4 files in standard storage.

## The Solution

### Native AWS Features

Initially, we explored AWS's native prefix-matching feature. However, due to the sheer volume of data and the intricate folder structure, this approach proved impractical. Manually listing folder names for such a large dataset would be inefficient and time-consuming.

### Custom Python Script

Recognizing the need for a tailored solution, we developed a Python script to automate the process. The script leveraged Boto3, the AWS SDK for Python, to interact with S3 programmatically.

### Iterative Refinement

While the initial script successfully handled objects outside folders and MP4 files, it encountered issues with files nested within folders. Through iterative refinement and debugging, we devised a robust solution capable of traversing nested structures and identifying files with the "RT" prefix, irrespective of their location.

## Script Logic

The script employed a simple yet effective logic flow:

1. List all objects within the S3 bucket.
2. For each object:
   - Check if it starts with the prefix "RT."
   - Determine its storage class.
   - If the object matches the prefix and is not already in Glacier, move it to Glacier Deep Archive.

## Script

```python
import boto3
import botocore

def list_s3_objects(bucket_name):
    """
    Lists all files (objects) in an S3 bucket, avoiding duplicate folder listings and extracting filenames.
    Additionally, changes the storage class of files with prefix "RT" to Glacier Deep Archive if not already in DEEP_ARCHIVE.

    Args:
        bucket_name (str): Name of the S3 bucket.
    """
    s3 = boto3.client('s3')
    paginator = s3.get_paginator('list_objects_v2')
    pagination_config = {'Bucket': bucket_name}

    seen_prefixes = set()  # Keep track of visited prefixes to avoid duplicates

    for page in paginator.paginate(**pagination_config):
        for content in page.get('Contents', []):
            key = content['Key']
            if not key.endswith('/'):  # Only process files
                filename = key.split("/")[-1]  # Extract filename
                if filename.startswith("RT"):
                    try:
                        object_info = s3.get_object(Bucket=bucket_name, Key=key)
                        current_storage_class = object_info.get('StorageClass', '')
                        print(current_storage_class)
                        if current_storage_class != 'DEEP_ARCHIVE':
                            # Change storage class to Glacier Deep Archive
                            s3.copy_object(
                                CopySource={'Bucket': bucket_name, 'Key': key},
                                Bucket=bucket_name,
                                Key=key,
                                StorageClass='DEEP_ARCHIVE'
                            )
                            print(f"Changed storage class of {filename} to Glacier Deep Archive")
                    
                    except botocore.exceptions.ClientError as e:
                        # Handle potential DEEP_ARCHIVE error by assuming it's DEEP_ARCHIVE
                        if e.response['Error']['Code'] == 'InvalidObjectState':
                            current_storage_class = 'DEEP_ARCHIVE'
                            print(f"{filename} is already in Glacier Deep Archive")
                        else:
                            raise e  # Re-raise other exceptions        
                
                else:
                    print(filename)  # Print filename for non-RT files
            else:
                prefix = key[:-1]  # Extract folder prefix without trailing slash
                if prefix not in seen_prefixes:
                    print("Folder:", prefix)  # Print folder prefix only once (if not seen before)
                    seen_prefixes.add(prefix)

bucket_name = "prod-notary-screen-records"
list_s3_objects(bucket_name)
```

## Note

If you're doing the same thing over and over again, this script could be better with some tweaks to make it work for those repeat tasks.

We proposed two strategies:

1. **Indexing and Parallel Processing:**
   - Index all S3 objects for faster retrieval.
   - Implement parallel batch processing by dividing the index into manageable chunks, enabling concurrent execution and improved efficiency.

2. **Using a DATE Variable:**
   - We can create a variable for the date and store it in an environment variable. Whenever a new object is added to the bucket, the script will run, check the date, and then move all objects added after that date to cold storage. Afterward, the variable storing the current date will be updated.

## Conclusion

By combining strategic thinking with technical expertise, we addressed our customer's challenge of optimizing S3 storage costs without compromising data accessibility. The custom Python script provided an efficient solution, demonstrating the power of automation and customization in cloud management. Whether it's a one-time migration task or an ongoing optimization initiative, leveraging AWS tools and custom scripts can streamline operations and drive cost savings in cloud storage management.
