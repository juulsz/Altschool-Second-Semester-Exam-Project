# My AWS EC2 Exam Project 

This project demonstrates hosting a dynamic landing page on an AWS EC2 instance using Nginx, Node.js, and Tailwind CSS, showcasing cloud computing and webhosting skills.

## Table of Contents
- [Project Overview](#project-overview)
- [URL](#url)
- [Steps I Took](#steps-i-took)
  - [Set Up EC2 Instance](#1-set-up-ec2-instance)
  - [Installed Nginx](#2-installed-nginx)
  - [Configured Node.js Reverse Proxy](#3-configured-nodejs-reverse-proxy-bonus)
  - [Deployed the Landing Page](#4-deployed-the-landing-page)
  - [HTTPS with Let's Encrypt](#5-https-with-lets-encrypt-theoretical-bonus)
- [Networking](#networking)
- [Screenshots](#screenshots)
- [Notes](#notes)
- [Conclusion](#conclusion)

## Project Overview
Built a static landing page using HTML and Tailwind CSS, hosted on an Ubuntu 24.04 AWS EC2 instance with Nginx. The page includes my name, role, project title, pitch, and bio. Nginx proxies '/api' requests to a Node.js app, and the site is accessible via the EC2 public IP (http://54.74.215.106) over HTTP.

## URL
`http://54.74.215.106`
Note: This is the public IP where the page is hosted.

## Steps I Took

### 1. Set Up EC2 Instance
- Signed into the AWS Management Console (https://aws.amazon.com/console/).
- Launched an EC2 instance:
  - Name: juliet-dev-server
  - OS: Ubuntu Server 24.04 LTS (Free Tier eligible)
  - Instance Type: 't2.micro' (Free Tier eligible)
  - Created and downloaded a key pair: 'juliet-key-1.pem'.
  - Configured a Security Group:
	 - HTTP (port 80): Anywhere (`0.0.0.0/0`)
	 - HTTPS (port 443): Anywhere (`0.0.0.0/0`)
	 - SSH (port 22) from my IP.
  - Assigned an Elastic IP for a static public IP (54.74.215.106).
- Connected via SSH using Termius with the '.pem' file and public IP.


### 2. Installed Nginx
- Updated packages and installed Nginx:
 	```bash
  	sudo apt update
  	sudo apt install nginx -y


•  Started and enabled Nginx:
	sudo systemctl start nginx
	sudo systemctl enable nginx

•  Confirmed Nginx was running by accessing http://54.74.215.106 (displayed the default Nginx page).


### 3. Configured Node.js Reverse Proxy (Bonus):

	 -   Installed Node.js and npm:
		```bash
		sudo apt install nodejs npm -y

	•  Created a Node.js app in ~/node-app/app.js:

		const http = require('http');
		const server = http.createServer((req, res) => {
  			res.writeHead(200, { 'Content-Type': 'text/plain' });
  			res.end('Welcome! This backend is up and running - powered by Juliet\n');
		});
		server.listen(3000, () => {
  		console.log('Node.js server running on port 3000');
		});

	•  Used PM2 to run the app:
		sudo npm install -g pm2
		cd ~/node-app
		pm2 start app.js
		pm2 save
		pm2 startup

	•  Configured Nginx to proxy /api to Node.js by editing /etc/nginx/sites-available/default:
		server {
   			listen 80;
    			server_name 54.74.215.106;

    			root /var/www/html;
    			index index.html;

    			location / {
        		try_files $uri $uri/ /index.html;
    		}

    		location /api {
        		proxy_pass http://localhost:3000;
        		proxy_http_version 1.1;
        		proxy_set_header Upgrade $http_upgrade;
        		proxy_set_header Connection 'upgrade';
        		proxy_set_header Host $host;
        		proxy_cache_bypass $http_upgrade;
    		}
   	}

	•  Reloaded Nginx after verifying the configuration:
		sudo nginx -t
		sudo systemctl reload nginx

	•  Verified the Node.js app at http://54.74.215.106/api.



### 4. Deployed the Landing Page

	 -  Created 'index.html' with Tailwind CSS:

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Future of Cloud-Powered Health Records</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .animate-fade-in { animation: fadeIn 1s ease-in-out; }
        @keyframes fadeIn { 0% { opacity: 0; transform: translateY(20px); } 100% { opacity: 1; transform: translateY(0); } }
 	h2:hover {
        	background-color: #3b82f6;
        	color: white;
        	transition: all 0.3s ease;
        	cursor: pointer;
        	padding: 0.25rem 0.5rem;
        	border-radius: 0.25rem;
    </style>
</head>
<body class="bg-gray-100 font-sans">
    <header class="bg-blue-900 text-white text-center py-12">
        <h1 class="text-4xl font-bold animate-fade-in">Juliet Eluaka</h1>
        <p class="text-xl mt-2">Lead Cloud Engineer</p>
    </header>
    <main class="max-w-4xl mx-auto p-6">
        <section class="bg-white rounded-lg shadow-md p-6 mb-6 animate-fade-in">
            <h2 class="text-2xl font-semibold text-blue-600">Project: The Future of Cloud-Powered Health Records</h2>
            <p class="mt-4">This project showcases a cloud-based health records platform hosted on AWS EC2, enabling secure, scalable storage and access to patient data. Its innovative use of cloud technology ensures real-time availability and robust encryption, improving healthcare delivery and patient outcomes.</p>
        </section>
        <section class="bg-white rounded-lg shadow-md p-6 animate-fade-in">
            <h2 class="text-2xl font-semibold text-blue-600">About Me</h2>
            <p class="mt-4">Hi, I'm Juliet. I'm a budding cloud engineer with a passion for building scalable systems. My skills include AWS, Linux, Web development, and Cybersecurity. I recently completed a project deploying a web server on EC2, and I'm pursuing a degree in Cloud Engineering at AltSchool.</p>
        </section>
    </main>
    <footer class="bg-blue-900 text-white text-center py-4">
        <p>&copy 2025 Juliet Eluaka. All rights reserved.</p>
    </footer>
</body>
</html>

	 -   Transferred the file to EC2 using gitbash on my local device:
		scp -i ~/Downloads/juliet-key-1.pem ~/Downloads/index.html ubuntu@54.74.215.106:/home/ubuntu/

	•  Moved the index.html file to the Nginx web root:
		sudo mv /home/ubuntu/index.html /var/www/html/index.html

	•  Set proper file permissions to ensure the file is readable by the web server:
		sudo chmod 644 /var/www/html/index.html

	•  Confirmed the page at `http://54.74.215.106`.



### 5. HTTPS with Let's Encrypt (Theoretical Bonus)

	•  Note: Let's Encrypt requires a registered domain for SSH certificate issuance, which was not used to avoid costs. Below are the steps I'd take to set it up:

		•  Install Certbot:
			```bash
			sudo apt install certbot python3-certbot-nginx -y

		•  Run Certbot to obtain and install an SSL certificate:
			sudo certbot --nginx

		•  Provide an email, agree to terms, select the domain, and configure HTTP-to-HTTPS redirection.

		•  Verify the site at https://<domain> and test certificate renewal:
			sudo certbot renew --dry-run

		•  Since no domain was used, the site runs on HTTP at `http://54.74.215.106`

### Networking
 -   Configured Security Group to allow:
	 -  SSH (port 22): My IP
	 -  HTTP (port 80): Anywhere (`0.0.0.0/0`)
	 -  HTTPS (port 443): Anywhere (`0.0.0.0/0`) for potential HTTPS

 - Assigned an Elastic IP (`54.74.215.106`) for a static public IP.




### Screenshots
	- Landing Page: Shows the rendered page at `http://54.74.215.106` (see `landing-page-http.png`).
	
	- Node.js Response: Shows the API response at `http://54.74.215.106/api` (see `node-api.png`)


### Notes
	-  Used AWS Free Tier to minimize costs.
	-  No HTTPS implemented due to the lack of a registered domain.

### Conclusion
This project enhanced my skills in AWS EC2 instance setup, EC2 management, Nginx configuration, and Node.js proxying. It also deepened my understanding of cloud-based web hosting and deployment, paving the way for future projects in cloud engineering.
