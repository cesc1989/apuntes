# Name Collision and Missing Files in Magic Processed Folder - Technical Overview

## Related

- [[MPF_Issues_and_Replication]]
- [[File_Uploads_Current_State_&_Diagrams]]

# Context

As therapists fill in their Therapist Signup form, the files they submit are stored in a folder in a AWS Bucket. This folder is named using the therapist’s ID from the therapist-signup database.

All files uploaded to the bucket trigger a lambda that makes a request to a Marketplace endpoint. This endpoint downloads each uploaded file, converts it to PDF or Google Doc, and upload all versions (original, pdf, doc) to Google Drive under a folder with the therapist’s name (i.e “Quintero, Francisco”). Finally, Success team browse these folders to complete the Therapists credentialing process.

## Issues

Multiple reports describe therapists files for their Magically Processed Folder being stored in a different therapist’s MPF folder. Other reports indicate files don’t reach the expected MPF folder.

These are some of the reports about files going to an unexpected MPF:

- *Nesting Doll: Katrina Abdullah, Shawn Abraham, and Ania Adamaszek > Erin Adams and Jario Abreu > Brooke Adler*
- *There is a nesting doll effect in the MPF of Marlou Acosta, it has the other MPF's for Abraham, Joel & Cunningham, Paul*
- *Nesting doll situation: All the files for Nicholas Lamb have been mixed in with the files for Nicholas Lambert in the Lambert folder*

## **Notes**

Files are present in the therapist’s folder in the AWS bucket meaning the sync is failing from Marketplace to the MPF.

**Important**: There is no MPF setup in Alpha.

# Debugging

## Inspecting Logs

Output in the logs hasn’t shown yet clear indication of what the error might be. This could be because grepping logs needs to be done the closest to the moment the error is reported or I’ve been looking in the wrong place.

I’ve found it difficult to inspect logs because of the way the Lambda logs are organized. However, the times I’ve been able to follow the track, the logs in the lambda show the request to the Marketplace endpoint is done with success.

*Note: See* [*attachment*](https://paper.dropbox.com/doc/Name-Collision-and-Missing-Files-in-Magic-Processed-Folder-Technical-Overview--CQfYFpGGcRlj68AgzUBTvMaUAg-HMJmryXAwB4kfXkwQvjSv#:uid=670242849822905117470348&h2=Attachments) *to know parts of the flow being logged.*

## Replicating the Issue Locally

In the local env, I wasn’t able to recreate the error. The only combination that produced a misplacing of files was with therapists having same first name and last name.

A concurrency issue, given the slow nature of local env, is difficult to reproduce. However, code related to create therapist’s folder in Google Drive is wrapped in a concurrency lock.

```python
   def fetch_or_create_folder_ids(folder_name: str, root_folder_id: str) -> List[str]:
        """Return all folder ids that have the given folder name, and create it if it does not exist yet."""
        with cache.new_distributed_lock(f"{root_folder_id}/{folder_name}", timeout=300, blocking_timeout=30):
            folder_found = find_folder_ids(root_folder_id, folder_name)
            if folder_found:
                return folder_found
            return create_folder(folder_name, root_folder_id)
```

## Sentry Errors

No errors have been caught by Sentry.

# Proposals

## Change therapist folder name format

The current format is:

    last_name, first_name

New suggested format:

    last_name_first_name_id

Some time ago, in the AWS Bucket, therapists folders had a similar pattern for the name and it caused multiple issues. Nowadays, therapists folder in AWS are named using the therapist’s ID (the one in the therapist-signup database) and folder name related issues no longer pop up.

Making sure folder names are always unique would remove it as the likeliness of the error.

## Add more logs

Identify additional parts of the flow (see [attachment](https://paper.dropbox.com/doc/Name-Collision-and-Missing-Files-in-Magic-Processed-Folder-Technical-Overview--CQfYFpGGcRlj68AgzUBTvMaUAg-HMJmryXAwB4kfXkwQvjSv#:uid=215342341836510774739809&h2=Logged-steps-in-the-flow-from-)) to add more logs.

## Request Marketplace Omega logs to be present in Omega

There’s already an ongoing effort from Infra team to make infra components available in Grafana. Grafana provides a better explorer tool to inspect and traverse logs (compared to AWS Cloud Watch) for any file upload to the MPF.
This request would be to have them prioritize Marketplace setup in Grafana.

# Attachments

## Logged steps in the flow from File in S3 to MPF

![](https://paper-attachments.dropboxusercontent.com/s_2F89114512C50EA0AC74EEA883C0C4910044AF78AD683BFBD31CE6DFE6833587_1717586904979_from.s3.to.drive.png)


