# How to list files per condition in S3 Bucket
[aws s3 ls reference](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/ls.html)

# List files per sizes in S3 Bucket

First need to run this command to get the list from the S3 Bucket:

    s3ls s3://luna-alpha-workloads-therapist-signup --recursive --summarize --human-readable

It returns a list of file names and their size.

Then, with one line, we can extract the file sizes only:

    s3ls s3://luna-alpha-workloads-therapist-signup --recursive --human-readable | grep MiB | awk '{ print $3, $4 }' | sort -h -r >> ~/Downloads/one_liner.txt

Command by command:

    s3ls s3://luna-alpha-workloads-therapist-signup --recursive --human-readable >> ~/Downloads/file_sizes_alpha_therapist.txt
    
    cat ~/Downloads/file_sizes_alpha_therapist.txt | grep MiB >> ~/Downloads/megabites.txt
    
    cat ~/Downloads/megabites.txt | awk '{ print $3, $4 }' >> ~/Downloads/processable.txt
    
    cat ~/Downloads/processable.txt | sort -h -r >> ~/Downloads/processable_sorted.txt

Notice I used commands:

- aws s3 ls
- grep
- awk
- sorth -h -r

Links:

- https://stackoverflow.com/q/43150572/1407371


# List files to see file extension

Get for one specific file field

    s3ls s3://luna-alpha-workloads-therapist-signup --recursive --human-readable | grep back_of_government_photo_id

Now invert the inspection:

    s3ls s3://luna-alpha-workloads-therapist-signup --recursive --human-readable | grep back_of_government_photo_id | awk '{ print $5, $3, $4 }'

Remove the uuid from the print:

    s3ls s3://luna-alpha-workloads-therapist-signup --recursive --human-readable | grep back_of_government_photo_id | awk '{ print $5 }' | sed 's|.*/||'

Ordered:

    s3ls s3://luna-alpha-workloads-therapist-signup --recursive --human-readable | grep back_of_government_photo_id | awk '{ print $5 }' | sed 's|.*/||' | sort

Group and count:

    s3ls s3://luna-alpha-workloads-therapist-signup --recursive --human-readable | grep back_of_government_photo_id | awk '{ print $5 }' | sed 's|.*/||' | sort | uniq -c


## For Omega
    s3ls s3://luna-omega-workloads-therapist-credentialing --recursive --human-readable | grep back_of_government_photo_id | awk '{ print $5 }' | sed 's|.*/||'

Full command:

    s3ls s3://luna-omega-workloads-therapist-credentialing --recursive --human-readable | grep back_of_government_photo_id | awk '{ print $5 }' | sed 's|.*/||' | sort | uniq -c

