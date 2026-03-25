## Why MPF? 🟢

> "It exists as an easy was for my team to access all of the submitted therapist documents but also house all of the therapists information completed during the credentialing and enrollment processes (primary source verifications, proof of insurance submissions and received approvals)."
> 
> Jessica Simon

> "It also exists because Ryan did not want to give everyone access to AWS from a security standpoint."
> 
> Jami Bragg

## The Journey of a File From Credentialing Application to Magically Processed Folder

1. Therapist uploads file in the Credentialing Application
2. Credentialing Application pushes file to AWS Bucket/[therapist_id] folder
3. AWS Lambda reading files in the bucket makes request to Marketplace endpoint
4. Marketplace endpoint triggers background job to upload file to Google Drive (MPF, same thing)
5. Background job downloads the file from the bucket, converts to multiple versions (doc, pdf), and uploads original + converted ones to MPF
6. Success...?
