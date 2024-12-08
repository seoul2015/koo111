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
from ftplib import FTP
import os

class FTPReceiver:
    def __init__(self, server, port, username, password, efs_base_path):
        self.server = server
        self.port = port
        self.username = username
        self.password = password
        self.efs_base_path = efs_base_path

    def download_to_efs(self, remote_file_path, file_name):
        # EFS 경로에 저장될 파일 경로 생성
        local_file_path = os.path.join(self.efs_base_path, file_name)

        # FTP 연결
        with FTP() as ftp:
            try:
                print(f"Connecting to FTP server {self.server}:{self.port}")
                ftp.connect(self.server, self.port)
                ftp.login(self.username, self.password)
                print("Connected successfully.")

                # 파일 다운로드
                with open(local_file_path, 'wb') as local_file:
                    ftp.retrbinary(f"RETR {remote_file_path}", local_file.write)
                print(f"File downloaded and saved to EFS: {local_file_path}")
            except Exception as e:
                print(f"Failed to download file: {e}")
```
---
