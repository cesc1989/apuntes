# Spike: File Uploads and Docker Containers - Therapist Signup
We want to make sure files users intent to upload are correctly stored in the S3 bucket. In this document I describe what we currently do, some limitations, and possible alternatives to make this file uploading process more reliable.

# Considerations
- File uploads are done in a synchronous way.
    - This is because application container and worker container have different filesystems.
    - Container file system is ephemeral. Cannot be trusted.
- We don’t use any third party library.
    - Integration done directly using S3 SDK because at the time the standard library for the framework didn’t support custom paths.
- The `public_url` value of the uploaded file is stored in the corresponding attribute in the record.
    - We do this as a way to validate that it was uploaded even though the file is not publicly accesible.
- When a request that includes files fails, sometimes the field gets assigned a wrong string text. The process discards any file-related attribute if:
    - They’re a plain old string.
    - There aren’t any actual contents in the corresponding field, i.e: `file=0`
    - There’s no file whatsoever.


## Q: Why make file uploads work in background?
- To speed up the process to the user while navigating the form
- Do post processing in a background job and then report back if issues are found
    - Do file resizing
    - Do format transformations
# Docker Filesystem: Ephemeral and Volumes

Docker [docs](https://docs.docker.com/storage/) about data persistence layer.

Currently, there are three options:

- [Volumes](https://docs.docker.com/storage/volumes/)
    - Managed by Docker.
    - Store data in the host system.
- [Bind mounts](https://docs.docker.com/storage/bind-mounts/)
    - Mounts a file or directory from the host machine into the container.
- [tmpfs mounts](https://docs.docker.com/storage/tmpfs/)
    - Store data in the host system memory.
    - Data is lost after the container is stopped or removed.
## Why Volumes?
- You can manage volumes using Docker CLI commands or the Docker API
- Volumes can be more safely shared among multiple containers
    - This would serve our case as the Omega env is deployed to several containers

Setting a Docker Volume could be the way to have a persistence layer to put files on before the background job can pick them up.

# Conclusions
- Enabling a Docker Volume won’t solve all file upload issues the form might experience
- Besides using a Docker Volume we need to run checks in frontend and backend regarding file size and type
    - We need to define acceptable values with Jessica and Jami
- A Docker Volume would speed up the navigation between sections and do post processing
    - We’d be able to do file uploads in a background job. Freeing the main request
    - We’d be able to run validations and checks in a background job and update the form in an async way

