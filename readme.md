```
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'software.amazon.awssdk:s3'
implementation 'org.apache.httpcomponents.client5:httpclient5:5.2'
```

```
@RestController
@RequestMapping("/api/files")
public class FileController {

    @PostMapping("/send")
    public ResponseEntity<String> sendFile(@RequestParam("file") MultipartFile file) {
        try {
            // 파일을 로컬 저장소 또는 임시 디렉토리에 저장
            Path tempDir = Files.createTempDirectory("temp");
            Path tempFile = tempDir.resolve(file.getOriginalFilename());
            file.transferTo(tempFile);

            // HTTPS로 파일 송신
            sendFileViaHttps(tempFile);

            // 임시 파일 삭제
            Files.delete(tempFile);

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
import tempfile
import requests

app = Flask(__name__)

@app.route('/api/files/send', methods=['POST'])
def send_file():
    try:
        # 파일 수신
        if 'file' not in request.files:
            return jsonify({"error": "No file provided"}), 400
        file = request.files['file']

        # 임시 디렉토리에 저장
        temp_dir = tempfile.mkdtemp()
        temp_file_path = os.path.join(temp_dir, file.filename)
        file.save(temp_file_path)

        # HTTPS로 파일 송신
        send_file_via_https(temp_file_path)

        # 임시 파일 삭제
        os.remove(temp_file_path)

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
