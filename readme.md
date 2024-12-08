```
import org.apache.commons.net.ftp.FTP;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;

public class EFSHttpSender {
    public static void main(String[] args) {
        String endpoint = "https://your-api-endpoint.com/upload";
        String data = "This is the data from EFS.";

        try {
            // Create URL object
            URL url = new URL(endpoint);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();

            // Configure connection
            connection.setRequestMethod("POST");
            connection.setDoOutput(true);
            connection.setRequestProperty("Content-Type", "application/json");
            connection.setRequestProperty("Accept", "application/json");

            // Send data
            try (OutputStream os = connection.getOutputStream()) {
                byte[] input = data.getBytes("utf-8");
                os.write(input, 0, input.length);
            }

            // Get response
            int responseCode = connection.getResponseCode();
            System.out.println("Response Code: " + responseCode);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
---

Python
---
```
import requests

def send_data_to_https():
    url = "https://your-api-endpoint.com/upload"
    data = {
        "content": "This is the data from EFS."
    }

    try:
        # Send POST request with JSON data
        response = requests.post(url, json=data)
        print(f"Response Code: {response.status_code}")
        print(f"Response Body: {response.text}")
    except Exception as e:
        print(f"An error occurred: {e}")

# Example usage
if __name__ == "__main__":
    send_data_to_https()
```
---
