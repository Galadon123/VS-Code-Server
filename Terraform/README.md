### **Terraform Configuration**

```py
provider "aws" {
  region = "us-east-1"
}

resource "aws_key_pair" "deployer" {
  key_name   = "deployer-key"
  public_key = file("~/.ssh/id_rsa.pub")
}

resource "aws_security_group" "vscode_sg" {
  name        = "vscode-sg"
  description = "Allow SSH and HTTP access"
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "vscode_server" {
  ami           = "ami-0c55b159cbfafe1f0" # Ubuntu 22.04 LTS (at the time of writing)
  instance_type = "t2.micro"
  key_name      = aws_key_pair.deployer.key_name
  security_groups = [aws_security_group.vscode_sg.name]

  user_data = <<-EOF
              #!/bin/bash
              curl -fsSL https://code-server.dev/install.sh | sh
              echo '[Unit]
              Description=VS Code Server
              After=network.target

              [Service]
              Type=simple
              User=ubuntu
              Environment="PASSWORD=105925"
              ExecStart=/usr/bin/code-server --bind-addr 0.0.0.0:8080 /home/ubuntu
              Restart=on-failure

              [Install]
              WantedBy=multi-user.target' | sudo tee /etc/systemd/system/code-server.service
              
              sudo systemctl daemon-reload
              sudo systemctl enable code-server
              sudo systemctl start code-server
            EOF

  tags = {
    Name = "VSCode-Server-Instance"
  }
}

output "instance_public_ip" {
  description = "The public IP of the EC2 instance"
  value       = aws_instance.vscode_server.public_ip
}
```

### **Explanation**

- **Provider**: Specifies the AWS region.
- **Key Pair**: Creates a key pair for SSH access using your local public key.
- **Security Group**: Allows inbound SSH (port 22) and HTTP (port 8080) traffic.
- **EC2 Instance**: 
  - Uses the Ubuntu 22.04 AMI.
  - Configures a `user_data` script to install VS Code Server, set up the systemd service, and start it.
- **Output**: Displays the public IP of the instance after creation.

### **Steps to Deploy**

1. **Initialize Terraform:**
   ```bash
   terraform init
   ```

2. **Apply the Terraform Configuration:**
   ```bash
   terraform apply
   ```

3. **Access VS Code Server:**
   - After the instance is created, get the public IP from the output.
   - Open a browser and go to `http://<instance-public-ip>:8080`.
   - Use the password `105925` to log in.
