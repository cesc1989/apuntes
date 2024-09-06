# Mitigate File-Upload Issues

# Context

Reports:

- *Users* report the files they uploaded are not present in the form.
- *Therapist team* report files are not present in the S3 folder or the Magically Processed folder.
- *Therapist team* have to download and convert some files because they are not synched from S3 folder to the Magically Processed folder.


# What do we know?

- Some users upload unexpected file types.
    - For example:
        - zip
        - md
        - jfif
        - txt (in a photo field)
        - nef
        - tif
        - exe (Windows executable)
        - eml
        - html
        - pptx
        - mp4
        - webarchive
        - pub
        - textClipping
        - crdownload (Chrome download progress)
        - bat (Windows executables)
- Some users upload [large](https://lunacare.atlassian.net/browse/LTC-630) files.
    - Largest file found: 310mb
    - Other large files:
        - 176mb
        - 93mb
        - 80mb


# What have we considered to do?

Idea: Limit the size of the files.
Hypothesis: In slow networks, large files might fail to upload.
Affects: Users credentialing process.

[[Plan_to_Determine_File_Size_Limits]]

Idea: Restrict file types to a smaller subset.
Hypothesis: Sync from S3 to Magically Processed folder discards some file types.
Affects: Therapist team need to manual convert some file types to be able to upload them to the Magically Processed folder.

[[File_Types_for_Existing_Uploaded_Files]]
