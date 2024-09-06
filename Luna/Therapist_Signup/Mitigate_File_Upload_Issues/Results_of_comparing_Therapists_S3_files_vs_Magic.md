# Results of comparing Therapists S3 files vs Magic Folder uploaded ones

This document outlines some of the findings after running a script to identify therapist files uploaded to the Magic Folder and then comparing against the ones present in their corresponding S3 folder.

# Details

The script was run to inspect all folders existing since June 1st, 2023.

    python -m marketplace.commands.google_drive list-folder-and-files-in-root-gdrive-folder --start 2023-06-01


## Attachments

- Inspection in S3 folders → [[S3_Files_-_Inspection]]
- Inspection in GDrive folders → [[GDrive_Files_-_Inspection]]

## Users the success team had to dig files in the S3 folder

- Chelsea Hamric
- Natalie Wek
- Noah Bernhardt
- John-Robert Woolley
- Carolynn Webb
- Nicholas Morales
- Vanesa Proto
- Tajon Dunn
- Scott Freedman
- Mikela Velasquez
- Ruei An Lin
- Christine Umali

## Users found in the list of inspected subfolders in the Magic Folder

- Tajon Dunn
- Mikela Velasquez
- Ruei An Lin
- Christine Umali



# Findings

**The script did not find all of the therapists reported by success team.**
Only four of the reported users where found in the script results. Could this be caused by the Service Accounts configuration?

**Files are being correctly uploaded to the S3 folder.**
As we can infer in [+S3 Files - Inspection](https://paper.dropbox.com/doc/S3-Files-Inspection-jnI2wPAeTnUMrSdZnE1In) where the regular amount of uploaded files (16) is present.

**There is no pattern for the missing file in GDrive.**
We can see in [+GDrive Files - Inspection](https://paper.dropbox.com/doc/GDrive-Files-Inspection-IkGAbcJBXNIxrCDkYBDKC) for therapists Dunn and Velasquez only some files are missing whilst Lin and Umali are missing many files.

This is also evident in the script results for the found therapists. For example, Nicholas Kaftan’s folder in GDrive only lists two files. In S3 there’s sixteen items.


# What could be done next?

**Trigger GDrive uploads by a Webhook and not a lambda.**
Instead of uploading each file as it’s pushed to the S3 folder, via Lambda, setup a webhook to be called once the form is submitted by the therapist.

This way the upload could be the whole batch of files. Not in a one by one basis.

**Identify who is failing: the lambda or the GDrive API call?**
Bearing in mind some GDrive folders contain a couple of files only, inspecting logs of the lambda and/or GDrive would help to identify what service is failing. Is it both?

