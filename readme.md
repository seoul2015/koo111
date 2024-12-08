```
<dependencies>
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>s3</artifactId>
        <version>2.20.2</version>
    </dependency>
</dependencies>
```

```
import software.amazon.awssdk.auth.credentials.ProfileCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.S3Client;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class S3Config {
    @Bean
    public S3Client s3Client() {
        return S3Client.builder()
                .region(Region.US_EAST_1) // 원하는 리전을 설정하세요
                .credentialsProvider(ProfileCredentialsProvider.create())
                .build();
    }
}

```

```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.model.PutObjectRequest;

import java.io.IOException;
import java.nio.file.Path;

@RestController
@RequestMapping("/api/files")
public class FileController {

    @Autowired
    private S3Client s3Client;

    @PostMapping("/upload")
    public String uploadFileToS3(@RequestParam("efsFilePath") String efsFilePath, 
                                 @RequestParam("bucketName") String bucketName, 
                                 @RequestParam("keyName") String keyName) {
        try {
            Path filePath = Path.of(efsFilePath);
            s3Client.putObject(PutObjectRequest.builder()
                    .bucket(bucketName)
                    .key(keyName)
                    .build(), filePath);
            return "File uploaded successfully to S3!";
        } catch (Exception e) {
            e.printStackTrace();
            return "Error uploading file: " + e.getMessage();
        }
    }
}

```
---

Python
---
```
from flask import Flask, request, jsonify
import boto3
import os

app = Flask(__name__)

# AWS S3 설정
s3_client = boto3.client(
    's3',
    aws_access_key_id='YOUR_ACCESS_KEY',
    aws_secret_access_key='YOUR_SECRET_KEY',
    region_name='us-east-1'
)

@app.route('/upload', methods=['POST'])
def upload_file_to_s3():
    efs_file_path = request.form.get('efsFilePath')
    bucket_name = request.form.get('bucketName')
    key_name = request.form.get('keyName')

    if not os.path.exists(efs_file_path):
        return jsonify({"error": "File does not exist on EFS"}), 400

    try:
        with open(efs_file_path, 'rb') as file_data:
            s3_client.upload_fileobj(file_data, bucket_name, key_name)
        return jsonify({"message": "File uploaded successfully to S3!"}), 200
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True)
```
---
