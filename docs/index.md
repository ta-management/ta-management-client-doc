---
hide:
  - navigation
  - toc
---

# Frontend Deployment Instruction
This guide assumes the backend deployment instructions were executed first.

## Step 1 — Install Node.js
#### Install nodejs and npm
`sudo apt install nodejs` 

## Step 2 — Setting up the Production Build
#### Navigate to the frontend folder from the root of the project directory
`cd frontend` 
#### Install Node Modules
`npm install`
#### Create the Production Build
`npm run build`

## Step 3 — Configuring the Deployment Location

#### Open a new server block in Nginx’s sites-available directory

`sudo nano /etc/nginx/sites-available/react`

#### Specify server block information

```
/etc/nginx/sites-available/react

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /home/ubuntu/ta-management/frontend/build;
    index index.html index.htm index.nginx-debian.html;
    
    server_name www.ua-compsci-ta.ca ua-compsci-ta.ca;

    location / {
        try_files $uri $uri/ = 404;
    }
}
```

#### Restart nginx
`sudo systemctl restart nginx`

## For updates to the React application
If you would like to deploy any changes made to the frontend files:

Navigate to the frontend directory

`npm run build`

`sudo systemctl restart nginx`

---

## Reference

[DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-react-application-with-nginx-on-ubuntu-20-04)