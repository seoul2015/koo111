```
import org.apache.commons.net.ftp.FTP;
import org.apache.commons.net.ftp.FTPClient;

import java.io.FileOutputStream;
import java.io.IOException;

public class FTPReceiver {
    private String server;
    private int port;
    private String user;
    private String password;
    private String efsBasePath; // EFS 마운트 경로

    public FTPReceiver(String server, int port, String user, String password, String efsBasePath) {
        this.server = server;
        this.port = port;
        this.user = user;
        this.password = password;
        this.efsBasePath = efsBasePath;
    }

    public void downloadToEFS(String remoteFilePath, String fileName) throws IOException {
        FTPClient ftpClient = new FTPClient();

        try {
            // FTP 서버 연결
            ftpClient.connect(server, port);
            ftpClient.login(user, password);

            // 파일 다운로드를 위한 설정
            ftpClient.enterLocalPassiveMode();
            ftpClient.setFileType(FTP.BINARY_FILE_TYPE);

            // EFS로 저장할 경로 생성
            String efsFilePath = efsBasePath + "/" + fileName;

            // 파일 다운로드
            try (FileOutputStream outputStream = new FileOutputStream(efsFilePath)) {
                boolean success = ftpClient.retrieveFile(remoteFilePath, outputStream);
                if (success) {
                    System.out.println("File downloaded successfully to EFS: " + efsFilePath);
                } else {
                    System.out.println("Failed to download the file.");
                }
            }
        } finally {
            // 연결 종료
            ftpClient.logout();
            ftpClient.disconnect();
        }
    }
}
```

---

Controller
---
```
import org.springframework.web.bind.annotation.*;
import java.io.IOException;

@RestController
@RequestMapping("/ftp")
public class FTPController {

    private final FTPReceiver ftpReceiver;

    public FTPController() {
        // FTP 서버 및 EFS 설정
        this.ftpReceiver = new FTPReceiver(
            "ftp.example.com", 
            21, 
            "username", 
            "password", 
            "/mnt/efs" // EFS 마운트 경로
        );
    }

    @PostMapping("/receive")
    public String receiveFile(
        @RequestParam String remoteFilePath,
        @RequestParam String fileName) {
        try {
            ftpReceiver.downloadToEFS(remoteFilePath, fileName);
            return "File successfully saved to EFS: " + fileName;
        } catch (IOException e) {
            e.printStackTrace();
            return "Failed to receive file: " + e.getMessage();
        }
    }
}
```
---


Python
---
```
from flask import Flask, request, jsonify
from ftplib import FTP
import os

app = Flask(__name__)

# EFS 및 FTP 설정
EFS_BASE_PATH = "/mnt/efs"
FTP_SERVER = "ftp.example.com"
FTP_PORT = 21
FTP_USERNAME = "your_username"
FTP_PASSWORD = "your_password"

def download_to_efs(remote_file_path, file_name):
    local_file_path = os.path.join(EFS_BASE_PATH, file_name)
    with FTP() as ftp:
        ftp.connect(FTP_SERVER, FTP_PORT)
        ftp.login(FTP_USERNAME, FTP_PASSWORD)
        with open(local_file_path, 'wb') as local_file:
            ftp.retrbinary(f"RETR {remote_file_path}", local_file.write)
    return local_file_path

@app.route("/ftp/receive", methods=["POST"])
def receive_file():
    try:
        data = request.json
        remote_file_path = data["remote_file_path"]
        file_name = data["file_name"]

        # 파일 다운로드 및 EFS 저장
        local_file_path = download_to_efs(remote_file_path, file_name)
        return jsonify({"message": f"File saved to EFS: {local_file_path}"})
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
---
