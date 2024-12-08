```
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'software.amazon.awssdk:s3'
implementation 'org.apache.httpcomponents.client5:httpclient5:5.2'
```

```
@RestController
@RequestMapping("/api/files")
public class FileController {

    // EFS 마운트된 경로 (예: /mnt/efs)
    private static final String EFS_PATH = "/mnt/efs";

    @PostMapping("/send")
    public ResponseEntity<String> sendFile(@RequestParam("file") MultipartFile file) {
        try {
            // EFS 경로에 파일 저장
            Path filePath = Paths.get(EFS_PATH, file.getOriginalFilename());
            file.transferTo(filePath);

            // HTTPS로 파일 송신
            sendFileViaHttps(filePath);

            return ResponseEntity.ok("File sent successfully.");
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Failed to send file.");
        }
    }

    private void sendFileViaHttps(Path filePath) throws Exception {
        HttpClient client = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
                .uri(new URI("https://example.com/upload"))
                .header("Content-Type", "application/octet-stream")
                .POST(HttpRequest.BodyPublishers.ofFile(filePath))
                .build();

        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        if (response.statusCode() != 200) {
            throw new RuntimeException("Failed to send file. Response: " + response.body());
        }
    }
}
```

---

Python
---
```
from flask import Flask, request, jsonify
import os
import requests

app = Flask(__name__)

# EFS 마운트된 경로 (예: /mnt/efs)
EFS_PATH = "/mnt/efs"

@app.route('/api/files/send', methods=['POST'])
def send_file():
    try:
        # 파일 수신
        if 'file' not in request.files:
            return jsonify({"error": "No file provided"}), 400
        file = request.files['file']

        # EFS 경로에 파일 저장
        file_path = os.path.join(EFS_PATH, file.filename)
        file.save(file_path)

        # HTTPS로 파일 송신
        send_file_via_https(file_path)

        return jsonify({"message": "File sent successfully."}), 200
    except Exception as e:
        return jsonify({"error": str(e)}), 500


def send_file_via_https(file_path):
    url = "https://example.com/upload"
    with open(file_path, 'rb') as f:
        response = requests.post(url, files={'file': f})
    if response.status_code != 200:
        raise Exception(f"Failed to send file. Response: {response.text}")


if __name__ == '__main__':
    app.run(port=5000)
```
---
