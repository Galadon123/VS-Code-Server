### **Setting Up VS Code Server on AWS EC2 (Ubuntu) with Systemd**

This document outlines the steps to install and configure `code-server` on an AWS EC2 instance running Ubuntu, using a systemd service to manage the server.

#### **1. Install `code-server`**

Download and install the latest version of `code-server`:

```bash
curl -fsSL https://code-server.dev/install.sh | sh
```

#### **2. Create the Systemd Service File**

1. **Open the Service File for Editing:**

   ```bash
   sudo nano /etc/systemd/system/code-server.service
   ```

2. **Add the Following Content:**

   ```ini
   [Unit]
   Description=VS Code Server
   After=network.target

   [Service]
   Type=simple
   User=ubuntu
   Environment="PASSWORD=105925"
   ExecStart=/usr/bin/code-server --bind-addr 0.0.0.0:8080 /home/ubuntu
   Restart=on-failure

   [Install]
   WantedBy=multi-user.target
   ```

3. **Save and Exit:**
   Save the changes (`Ctrl + X`, then `Y`, and `Enter`).

#### **3. Reload Systemd and Start the Service**

1. **Reload the Systemd Daemon:**

   ```bash
   sudo systemctl daemon-reload
   ```

2. **Start and Enable the Service:**

   ```bash
   sudo systemctl start code-server
   sudo systemctl enable code-server
   ```

3. **Check the Status:**

   ```bash
   sudo systemctl status code-server
   ```

#### **4. Accessing VS Code Server**

- Open your browser and go to `http://<your-ec2-public-ip>:8080`.
- Use the password `105925` to log in.

---

This setup will automatically start `code-server` on boot and allow you to manage it using systemd commands.