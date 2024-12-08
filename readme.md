```
server.port=8080
efs.path=/efs/your_directory
```

```
import org.springframework.web.bind.annotation.*;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;

@RestController
@RequestMapping("/api")
public class EFSApiController {

    private static final String EFS_DIRECTORY = "/efs/your_directory/";

    @PostMapping("/upload")
    public String uploadData(@RequestBody String data) {
        try {
            // Define the file path in EFS
            File file = new File(EFS_DIRECTORY + "uploaded_data.txt");

            // Write data to the file
            try (FileWriter writer = new FileWriter(file)) {
                writer.write(data);
            }

            return "Data successfully saved to EFS!";
        } catch (IOException e) {
            e.printStackTrace();
            return "Failed to save data: " + e.getMessage();
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

app = Flask(__name__)

# EFS Directory Path
EFS_DIRECTORY = "/efs/your_directory"

@app.route('/upload', methods=['POST'])
def upload_data():
    try:
        # Get the JSON data from the request
        data = request.get_json()
        if not data:
            return jsonify({"error": "No data provided"}), 400

        # Create file path
        file_path = os.path.join(EFS_DIRECTORY, "uploaded_data.txt")

        # Write data to the file
        with open(file_path, 'w') as file:
            file.write(str(data))

        return jsonify({"message": "Data successfully saved to EFS!"}), 200
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```
---
