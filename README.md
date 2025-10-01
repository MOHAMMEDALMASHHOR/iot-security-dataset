# iot-security-dataset
Staj 2
# IoT Security Dataset Generation Project

This project provides a reproducible lab environment using Docker to simulate a small IoT network. It includes services like an MQTT broker, a web API for a smart plug, and an RTSP camera stream. The primary goal is to generate and capture both normal and malicious network traffic to create a labeled dataset for academic research in IoT security, such as intrusion detection systems.



## Features
- **Containerized Environment:** Uses Docker Compose for easy setup and teardown.
- **Realistic IoT Services:** Includes MQTT, HTTP API, and RTSP services.
- **Normal Traffic Simulation:** Python scripts simulate legitimate sensor data and user interaction.
- **Attack Simulation:** A guided set of commands for port scanning, brute-force, and DoS attacks.
- **Data Capture & Labeling Pipeline:** Tools to convert raw packet captures (`.pcap`) into a labeled CSV dataset.

---

## Requirements
- **Docker:** [Docker Desktop](https://www.docker.com/products/docker-desktop/) for Windows/Mac or Docker Engine for Linux.
- **Python 3:** For running traffic generator and data processing scripts.
- **Kali Linux:** A separate machine (virtual or physical) for launching attacks.
- **Wireshark (Recommended):** For live traffic inspection and capture.

## 1. Lab Setup and Execution

### Step 1: Clone or Download the Project
Download all the project files and organize them into the specified folder structure.

### Step 2: Start the Docker Environment
Open a terminal in the root directory of the project (`iot-security-dataset/`) and run:
```bash
docker-compose up -d
```
This command will build the custom `http_api` image and start all containers in the background.

To check if all containers are running, use:
```bash
docker-compose ps
```
You should see all four services (`mosquitto`, `nodered`, `http_api`, `rtsp_camera`) with a `healthy` or `running` status.



### Step 3: Verify Services
- **Node-RED:** Open your web browser to `http://localhost:1880`. You should see the Node-RED flow editor.
- **HTTP API:** Open `http://localhost:5000/status` in your browser or use `curl`. You should get a JSON response like `{"status":"OFF", ...}`.
- **MQTT:** The broker is running but has no UI. We'll interact with it using our script.
- **RTSP Camera:** You can view the stream using a player like VLC. Open a network stream with the URL: `rtsp://localhost:8554/mystream`.

---

## 2. Generating Normal Traffic

Run these scripts from your host machine's terminal (not inside a container). They will connect to the exposed Docker ports.

First, install the required Python libraries:
```bash
pip install paho-mqtt requests
```

**1. Start the MQTT Publisher:**
This script simulates a sensor sending temperature data every 2 seconds.
```bash
python traffic_generators/mqtt_publisher.py
```

**2. Start the HTTP Client:**
This script simulates a user checking and toggling the smart plug every 5 seconds.
```bash
python traffic_generators/http_client.py
```
Let both scripts run. This is your "normal" background traffic.



---

## 3. Capturing Traffic

While the normal traffic generators are running, you need to capture all network packets.

#### Finding the Docker Host IP
Your Kali machine needs the IP address of the machine running Docker. On the Docker host, run `ip addr` (Linux/Mac) or `ipconfig` (Windows) to find the local network IP (e.g., `192.168.1.10`). This is your `TARGET_IP`.

#### Capturing with `tcpdump` (Linux/Mac)
Run this command on your Docker host machine to capture all traffic going to the IoT services and save it to a file.
```bash
sudo tcpdump -i any -w dataset/capture.pcap 'port 1883 or port 5000 or port 1880 or port 8554'
```
- `-i any`: Listen on all network interfaces.
- `-w dataset/capture.pcap`: Write the output to a file.
- `'port ...'`: A filter to only capture traffic relevant to our services.

Let this capture run in the background. **Start it before you run the attacks.**



---

## 4. Running the Attacks

Now, from your **Kali Linux machine**, follow the instructions in **`attack_commands.md`**.

For each attack:
1.  **Execute the command.**
2.  **Carefully log the start and end times.** This is crucial for labeling.
3.  **Use your Kali IP as the attacker IP** and the Docker host IP as the target IP.

After you have performed all attacks, stop the `tcpdump` process with `Ctrl + C`. You now have a `capture.pcap` file containing both normal and attack traffic.

---

## 5. Processing and Labeling the Dataset

This pipeline converts your raw `.pcap` file into a labeled `.csv` file.

### Step 1: Convert PCAP to CSV
First, install the `scapy` library:
```bash
pip install scapy
```
Run the conversion script:
```bash
python capture_and_label/pcap_to_csv.py dataset/capture.pcap dataset/unlabeled_traffic.csv
```
This will create a CSV file with extracted packet features.

### Step 2: Auto-Label the CSV
Now, use the `auto_labeler.py` script to add the 'label' column. You must provide the details of **one attack at a time**.

First, install the `pandas` library:
```bash
pip install pandas
```

**Example for the Hydra Brute Force attack:**
Assume your Kali IP is `192.168.1.50`, your Docker host is `192.168.1.10`, and the attack ran from `15:30:00` to `15:35:00` on September 30, 2025.

```bash
python capture_and_label/auto_labeler.py dataset/unlabeled_traffic.csv dataset/labeled_traffic.csv 192.168.1.50 192.168.1.10 '2025-09-30T15:30:00' '2025-09-30T15:35:00'
```
This command reads the unlabeled CSV, applies the "attack" label to all packets that match the time, source, and destination IPs, and saves a new labeled CSV file. You can re-run this command on the output of the previous run to label multiple attack scenarios in the same file.

### Example Dataset Preview

Your final `labeled_traffic.csv` will look like this:

| timestamp           | src_ip      | dst_ip      | src_port | dst_port | protocol | length | label  |
|---------------------|-------------|-------------|----------|----------|----------|--------|--------|
| 1664543401.123456   | 192.168.1.12| 192.168.1.10| 54321    | 1883     | TCP      | 102    | normal |
| 1664543402.567890   | 192.168.1.50| 192.168.1.10| 12345    | 5000     | TCP      | 60     | attack |
| 1664543402.567990   | 192.168.1.10| 192.168.1.50| 5000     | 12345    | TCP      | 54     | attack |
| 1664543403.223344   | 192.168.1.15| 192.168.1.10| 60123    | 5000     | TCP      | 88     | normal |



---

## Academic Usage

This generated dataset can be used to train and evaluate machine learning models for Intrusion Detection Systems (IDS) in IoT networks. The features can be expanded (e.g., using a tool like Zeek/Bro) for more complex analysis. When publishing, please cite the tools used (Docker, Nmap, Hydra, Scapy) and describe the lab setup methodology detailed in this README.