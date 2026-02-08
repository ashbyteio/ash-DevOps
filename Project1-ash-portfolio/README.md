## Project1: A static webpage hosted on AWS infra

## Basic specifications:
```
> A domain redirects to a webpage about author
> It will run on Flask/Nginx
> It will have an SSL certificate for secure connection
> Bonus: It will have a visitor counter
```

## To test on local setup: 

1. Install Flask using python virtual environment 
```bash
source .venv/bin/activate
pip3 install -r requirements.txt
```

2. Run the application:
```bash
python app.py
```
3. Access the site at `http://localhost:8000`

## EC2 Deployment

### Prerequisites
- EC2 instance with Amazon Linux
- Public IP/Internet GW/Security group (with port 80/8000/22/443) attached to the EC2 instance
- Python3/Git installed
```bash
sudo yum install git -y
sudo yum install python3 python3-pip -y 
```

### Setup Steps: 

#### Use Flask as the only web server: (Development grade)
1. Connect to EC2 instance and clone your public repo
```bash
git clone https://github.com/ashbyteio/ash-DevOps.git 
```

2. Update executable permissions to run linux-apps.sh and app.py

3. Comment out nginx install line in requirement.txt
```
# run nginx
# nginx -g "daemon off;"
```

4. Install Flask, dunicorn linux packages using
```bash
./linux-apps.sh
```

5. **Run the Flash app:**
```bash
python3 app.py
```

6. Access the site at `http:your-public-ip:8000`

#### Use Nginx as reverse proxy server (Production grade)

1. Add nginx package details in requirements.txt
```bash
# run nginx
nginx -g "daemon off;"
```
2. Install nginx package using 
```bash
./linux-apps.sh
```
3. Configure Nginx:
```bash
sudo vim /etc/nginx/conf.d/portfolio.conf
```
Add:
```
server {
    listen 80;
    server_name ec2-public-ip;  # domain name

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

}
```
4. **Start Nginx:**
```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```
5. Access the site at `http:your-public-ip`


### Security Group Configuration

Make sure EC2 security group allows:
- Port 22 (SSH)
- Port 80 (HTTP) from anywhere (0.0.0.0/0)
- Port 8000 if accessing Flask directly (or remove if using Nginx)

## Troubleshooting

1. **Port already in use:**
```bash
sudo lsof -i :5000
sudo kill -9 
```

2. **Permission issues:**
```bash
chmod +x app.py
```

## SSL Cert installation using Certbot for Secure connection
(AWS Cert manager does not issue SSL Cert for EC2 instances on a free tier account)

### Before getting the cert, get domain name ready and assign an "A" record to EC2's public IP in Route53 service of AWS. 

1. Install dependencies:
```bash
source .venv/bin/activate
sudo dnf install -y python3-pip augeas-libs
sudo python3 -m pip install certbot certbot-nginx
```
2. Update Nginx Config:
```bash
sudo nano /etc/nginx/conf.d/portfolio.conf
```
Add:
```
server {
    listen 80;
    server_name ashbyteio.xyz;  # Update domain here

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
3. Test and Restart Nginx:
```bash
sudo nginx -t
sudo systemctl restart nginx
```
4. Get SSL Certificate with Certbot (It will automatically configure Nginx)
```bash
sudo certbot --nginx -d ashbyteio.xyz
```

5. Follow the prompts to enter the email (optional) and agree to terms.
6. Update Security group to allow HTTPS traffic on Port 443 from 0.0.0.0/0
7. Verify secure site access: https://ashbyteio.xyz




