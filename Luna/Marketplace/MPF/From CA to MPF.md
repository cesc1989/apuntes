## The Journey of a File From Credentialing Application to Magically Processed Folder

1. Therapist uploads file in the Credentialing Application
2. Credentialing Application pushes file to AWS Bucket/[therapist_id] folder
3. AWS Lambda reading files in the bucket makes request to Marketplace endpoint
4. Marketplace endpoint triggers background job to upload file to Google Drive (MPF, same thing)
5. Background job downloads the file from the bucket, converts to multiple versions (doc, pdf), and uploads original + converted ones to MPF
6. Success...?