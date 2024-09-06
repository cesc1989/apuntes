# Plan to Determine File Size Limits
We want to ensure all files are properly uploaded. To do this we should consider how to handle file sizes. Should we limit uploads up to a defined size? Do we understand how large are files uploaded by users?

Let’s try to figure this out to have a clear path to follow.

# Considerations
## Notices
- We do not do any post processing to the files
- Files are uploaded synchronously
- Files are stored in an specific folder per user in the S3 bucket


## Files per section

These are the fields that hold files in the Credentialing Form:

**Signup**

- government_photo_id
- back_of_government_photo_id
- resume
- basic_life_support_certificate

**Personal Information**

- visa_or_green_card_proof
- photo

**Information for Credentialing**

- physical_therapy_diploma
- malpractice_liability_insurance_coverage
- direct_access_license
- residency_program_proof
- fellowship_program_proof
- board_certification_copy

**Medicare Requirement**

- support_file


> **NOTE**: in the Medicare Requirement section every answers has an optional file upload. This means the only field to reference a file is `support_file` but there could be multiple `support_file`s.

**Payout**

- w9_format
- business_irs_letter

**Certification and Release**

- handwritten_signature


# How large are files uploaded by users?

Let’s determine how large are files hosted in the S3 bucket for each possible upload.


# Do file size is currently causing problems?

Let’s test if file size causes issues in the form. Let’s try large files (up to 100mb) in normal connections and in slow connections. Also let’s try large file uploads in a mobile browser.

Test plan looks like this:


- Desktop browsers
    - Upload a 50mb file
        - Good connection
        - Slow connection
    - Upload a 100mb file
        - Good connection
        - Slow connection
- Mobile browsers (in app browser or native)
    - Upload a 50mb file
        - Good connection
        - Slow connection
    - Upload a 100mb file
        - Good connection
        - Slow connection

