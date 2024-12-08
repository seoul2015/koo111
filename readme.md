```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```
import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;

import java.util.Map;

@RestController
@RequestMapping("/upload")
public class UploadController {

    @PostMapping
    public ResponseEntity<?> receiveData(@RequestBody Map<String, Object> data) {
        try {
            System.out.println("Received Data: " + data);

            // 필요한 처리를 하고 응답 반환
            return ResponseEntity.ok(Map.of("status", "success", "received", data));
        } catch (Exception e) {
            e.printStackTrace();
            return ResponseEntity.status(500).body(Map.of("status", "error", "message", e.getMessage()));
        }
    }
}
```
---

Python
---
```
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/upload', methods=['POST'])
def upload_data():
    try:
        # JSON 데이터를 가져오기
        data = request.get_json()
        print(f"Received Data: {data}")

        # 필요한 처리를 하고 응답 반환
        return jsonify({"status": "success", "received": data}), 200
    except Exception as e:
        print(f"Error: {e}")
        return jsonify({"status": "error", "message": str(e)}), 500

if __name__ == '__main__':
    # HTTPS로 실행하기 위해 인증서와 키 설정
    app.run(host='0.0.0.0', port=5000, ssl_context=('cert.pem', 'key.pem'))
```
---
