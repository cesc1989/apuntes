# How to verify and make sure files were properly uploaded?

## AWS S3 docs links
- Client#put_object - [link](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/S3/Client.html#put_object-instance_method)
- Object#put - [link](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/S3/Object.html#put-instance_method)


# Situation

From time to time, files selected to be uploaded by therapist fail to do so and Sentry does not capture any kind of errors.

We have basic logging so trying to debug it that way could be very hard to do. I asked ChatGPT what I could I do and this is what it told me:

> 1) Implement a logging system that records when a file is uploaded to S3, and the result of the upload. This can give you an idea of whether there are any errors or issues occurring during the upload process.
> 
> 2)You can implement checksum validation to ensure that the file being uploaded is complete and not corrupted. You can calculate the MD5 checksum of the file on the client-side before uploading and compare it with the checksum of the file that gets uploaded to S3.

It also proposed comparing file size after uploading. Get the local file size and then compare it against the one uploaded. However this could be incorrect and pointed again to do the checksum.

    File.open(file_path, 'rb') do |file|
      resp = s3.put_object(
        body: file,
        bucket: bucket_name,
        key: file_name
      )
    
      # Get the size of the uploaded object
      s3_object = Aws::S3::Object.new(bucket_name, file_name, client: s3)
      s3_object_size = s3_object.content_length
    
      # Get the size of the local file
      local_file_size = File.size(file_path)
    
      # Check if the sizes match
      if s3_object_size == local_file_size
        puts "File #{file_name} uploaded successfully to bucket #{bucket_name}."
      else
        puts "Failed to upload file #{file_name} to bucket #{bucket_name}."
      end
    end


> Note that this approach assumes that the file was uploaded completely and that there were no errors during the upload process. If you want to ensure that the file was uploaded without errors, you should also consider implementing a checksum or hash of the file contents and comparing it with the checksum or hash of the uploaded object in S3.

Here’s the example for the checksum

    File.open(file_path, 'rb') do |file|
      # Calculate the MD5 checksum of the local file
      local_file_md5 = Digest::MD5.hexdigest(file.read)
    
      resp = s3.put_object(
        body: file,
        bucket: bucket_name,
        key: file_name
      )
    
      # Get the MD5 checksum of the uploaded object
      s3_object = Aws::S3::Object.new(bucket_name, file_name, client: s3)
      s3_object_md5 = s3_object.etag.delete('"')
    
      # Compare the checksums
      if local_file_md5 == s3_object_md5
        puts "File #{file_name} uploaded successfully to bucket #{bucket_name}."
      else
        puts "Failed to upload file #{file_name} to bucket #{bucket_name}."
      end
    end

Related doc to generate an [MD5 checksum](https://paper.dropbox.com/doc/Lecciones-Ruby-Therapists--B1bt08pmh_rqZSszLGCS2kM1Ag-bVYJG7QvkhoMPL2nletoG#:uid=310905295583679480030823&h2=Creando-un-Digest-MD5-para-arc).

    digest = Digest::MD5.file(@folder_name).base64digest
    @s3_client.put_object(bucket: @bucket, key: "#{@folder_name}/", content_md5: digest)

This is what the Client#put_object says:

> To ensure that data is not corrupted traversing the network, use the `Content-MD5` header. When you use this header, Amazon S3 checks the object against the provided MD5 value and, if they do not match, returns an error. Additionally, you can calculate the MD5 while putting an object to Amazon S3 and compare the returned ETag to the calculated MD5 value.


## Generating an MD5 checksum

what's the differente between:

    Digest::MD5.hexdigest(FILE)
    
    # and
    
    Digest::MD5.file(FILE).base64digest

According to ChatGPT:

> The difference between `Digest::MD5.hexdigest(FILE)` and `Digest::MD5.file(FILE).base64digest` is in the format of the digest that is returned.
> 
> `Digest::MD5.hexdigest(FILE)` returns the MD5 hash of the file **in hexadecimal format**, which is a string of 32 characters that represents the 128-bit hash value as a sequence of 16 bytes in hexadecimal notation.
> 
> `Digest::MD5.file(FILE).base64digest` returns the MD5 hash of the file in Base64 format, which is a string of 24 characters that represents the 128-bit hash value as a sequence of 16 bytes in a compact, URL-safe format.
> 
> In summary, both methods calculate the MD5 hash of a file, but `hexdigest` returns the hash value in hexadecimal format, while `base64digest` returns the hash value in Base64 format. Which method to use depends on the requirements of your application and how you want to represent the hash value.

