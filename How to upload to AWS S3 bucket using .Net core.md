This guide assumes you have access to an AWS account with subscription to S3 service and a bucket set up.

1.	Create and setup project
2.	Add a service
3.	Create a bucket
4.	Add security checks
5.	Upload a file
6.	Retrieve public URL for uploaded content
7.  Inject dependencies into startup file
8.  Add service to controller

Using an IDE of choice, create a new web project. It could be an API or a web app. For this demo, we will be creating an API using .Net 5.0.

## Create the API project
 Using Visual Studio or any IDE of your choice, create an API project. Choose .Net 5.0 as the target. Install the following nuget packages: `AWSSDK.S3` and `AWSSDK.Extensions.NETCore.Setup`

 ## Add a service
 Create a folder or class library project (depending on your preference) named Services; this will store our AWS service which will be called by the API controller.

 ##	Create a bucket
 To create a bucket, we will need to [connect to our AWS account with valid credential](https://docs.aws.amazon.com/sdk-for-net/latest/developer-guide/net-dg-config-netcore.html) using the nuget package AWSSDK.Extensions.NETCore.Setup. The nuget package “AWSSDK.S3” provides helpful classes for interacting with our upstream bucket. These classes will enable us perform actions such as creating and updating a bucket. Now, let us create a method that will create a bucket with a specified bucket name. This method will check if the bucket exists and create it if it doesn’t. Using `AmazonS3Client`, the bucket will be created using a `PutBucketRequest` object, containing the bucket information.

 ```
 public async Task<bool> CreateBucketAsync(string bucketName)
        {
            try
            {
                _logger.LogInformation("Creating Amazon S3 bucket...");
                var bucketExists = await AmazonS3Util.DoesS3BucketExistV2Async(_amazonS3Client, bucketName);
                if (bucketExists)
                {
                    _logger.LogInformation($"Amazon S3 bucket with name '{bucketName}' already exists");
                    return false;
                }

                var bucketRequest = new PutBucketRequest()
                {
                    BucketName = bucketName,
                    UseClientRegion = true
                };

                var response = await _amazonS3Client.PutBucketAsync(bucketRequest);

                if (response.HttpStatusCode != HttpStatusCode.OK)
                {
                    _logger.LogError("Something went wrong while creating AWS S3 bucket.", response);
                    return false;
                }

                _logger.LogInformation("Amazon S3 bucket created successfully");
                return true;
            }
            catch (AmazonS3Exception ex)
            {
                _logger.LogError("Something went wrong", ex);
                throw;
            }
        }
 ```

 ## Add security checks
As is the case with arbitrary file uploads by users, data is untrusted, hence, it must be checked to ensure it is clean and conforms to business requirements. For this demo, we will be requiring users to upload only image files (".jpg", ".jpeg", ".png", “gif”) not more than 6Mb. Furthermore, the file will be saved, not with the original file name, but a random name; the original name will be saved as part of the file meta. This will prevent injection and related malicious attacks. 

```
private bool IsValidImageFile(IFormFile file)
        {

            // Check file length
            if (file.Length < 0)
            {
                return false;
            }

            // Check file extension to prevent security threats associated with unknown file types
            string[] permittedExtensions = new string[] { ".jpg", ".jpeg", ".png", ".pdf" };
            var ext = Path.GetExtension(file.FileName).ToLowerInvariant();
            if (string.IsNullOrEmpty(ext) || !permittedExtensions.Contains<string>(ext))
            {
                return false;
            }

            // Check if file size is greater than permitted limit
            if (file.Length > _config.FileSize) // 6MB
            {
                return false;
            }

            return true;
        }
```

## 	Upload a file
To upload a file, the file must be represented as a `TransferUtilityUploadRequest` object. This object contains several properties, notably:
-	InputStream: a stream of the file content to be uploaded

- Key: The storage name for the file. This will be set to the random file name

- BucketName: specifies the destination bucket for upload

- CannedACL: specifies the access control policy for the uploaded file. This will be set to `S3CannedACL.PublicRead` so that users will be able to view the uploaded content via the generated link

-	MetaData: contains arbitrary information about the file. We will add the orginal file name as part of the metadata

With the upload object constructed, we can call `UploadAsync` method of `TransferUtility` class, passing it as a parameter. This will asynchronously trigger the upload process to AWS S3.

```
public async Task<AWSUploadResult<string>> UploadImageToS3BucketAsync(UploadRequestDto requestDto)
        {
            try
            {
                var file = requestDto.File;
                string bucketName = requestDto.BucketName;

                if (!IsValidImageFile(file))
                {
                    _logger.LogInformation("Invalid file");
                    return new AWSUploadResult<string>
                    {
                        Status = false,
                        StatusCode = StatusCodes.Status400BadRequest
                    };
                }

                // Rename file to random string to prevent injection and similar security threats
                var trustedFileName = WebUtility.HtmlEncode(file.FileName);
                var ext = Path.GetExtension(file.FileName).ToLowerInvariant();
                var randomFileName = Path.GetRandomFileName();
                var trustedStorageName = "files/" + randomFileName + ext;

                // Create the image object to be uploaded in memory
                var transferUtilityRequest = new TransferUtilityUploadRequest()
                {
                    InputStream = file.OpenReadStream(),
                    Key = trustedStorageName,
                    BucketName = bucketName,
                    CannedACL = S3CannedACL.PublicRead, // Ensure the file is read-only to allow users view their pictures
                    PartSize = 6291456
                };

                // Add metatags which can include the original file name and other decriptions
                var metaTags = requestDto.Metatags;
                if (metaTags != null && metaTags.Count() > 0)
                {
                    foreach (var tag in metaTags)
                    {
                        transferUtilityRequest.Metadata.Add(tag.Key, tag.Value);
                    }
                }

                transferUtilityRequest.Metadata.Add("originalFileName", trustedFileName);


                await _transferUtility.UploadAsync(transferUtilityRequest);

                // Retrieve Url
                var ImageUrl = GenerateAwsFileUrl(bucketName, trustedStorageName).Data;

                _logger.LogInformation("File uploaded to Amazon S3 bucket successfully");
                return new AWSUploadResult<string>
                {
                    Status = true,
                    Data = ImageUrl
                };
            }
            catch (Exception ex) when (ex is NullReferenceException)
            {
                _logger.LogError("File data not contained in form", ex);
                throw;
            }
            catch (Exception ex) when (ex is AmazonS3Exception)
            {
                _logger.LogError("Something went wrong during file upload", ex);
                throw;
            }

        }
```

##	Retrieve public URL for uploaded content
Additionally, we need a way to get a sharable URL which can be saved to a database. AWS has two patterns for constructing S3 file URLs, namely: Path style, which is deprecated and virtual hosted style. For this demo, we will use the virtual hosted style to retrieve the file URL. It follows any of the underlisted patterns
-	http://[bucketName].[regionName].amazonaws.com/[key]
-	https://[bucketName].s3.amazonaws.com/[key]

The full class for the upload service is shown below:

```
public AWSUploadResult<string> GenerateAwsFileUrl(string bucketName, string key, bool useRegion = true)
        {
            // URL patterns: Virtual hosted style and path style
            // Virtual hosted style
            // 1. http://[bucketName].[regionName].amazonaws.com/[key]
            // 2. https://[bucketName].s3.amazonaws.com/[key]

            // Path style: DEPRECATED
            // 3. http://s3.[regionName].amazonaws.com/[bucketName]/[key]
            string publicUrl = string.Empty;
            if (useRegion)
            {
                publicUrl = $"https://{bucketName}.{_config.AwsRegion}.{_config.AwsS3BaseUrl}/{key}";
            }
            else
            {
                publicUrl = $"https://{bucketName}.{_config.AwsS3BaseUrl}/{key}";
            }
            return new AWSUploadResult<string>
            {
                Status = true,
                Data = publicUrl
            };
        }
```

## Inject dependencies into startup file
Next, here is the code to inject dependencies which the service will need:

```
            // Add app injections
            services.AddDefaultAWSOptions(Configuration.GetAWSOptions());
            services.AddAWSService<IAmazonS3>();
            services.AddTransient<IUploadService, UploadService>();
            services.AddTransient<TransferUtility>();
```

These should be added to the `ConfigureServices` method.

## Add service to controller

Finally, we are ready to setup our controller. It will contain two endpoints; one for creating the bucket and another for uploading contents

``` 
        [HttpPost]
        public async Task<IActionResult> Post([FromForm] UploadRequestDto requestDto)
        {
            var result = await _uploadService.UploadImageToS3BucketAsync(requestDto);
            return StatusCode(result.StatusCode);
        }

        [HttpPost("create-bucket")]
        public async Task<IActionResult> CreateS3BucketAsync(string bucketName)
        {
            await _uploadService.CreateBucketAsync(bucketName);
            return StatusCode(StatusCodes.Status200OK);
        }
```

For the full implementation, please [visit this repo](https://github.com/samtimberlan/AWSService)

There you have it. Thanks for reading.

Did you spot a typo, an error or want to contribute? [Here's the repo on GitHub](https://github.com/samtimberlan/Blog-Posts/blob/drafts/How%20to%20upload%20to%20AWS%20S3%20bucket%20using%20.Net%20core.md)