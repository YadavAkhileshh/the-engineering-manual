# 20 Backend Deployment

This guide covers backend hosting environments: virtual private servers (VPS), serverless functions, PaaS options, Nginx reverse proxies, load balancers, HTTPS automation, and zero-downtime releases.

## Backend Hosting Fundamentals

### Definition
Backend hosting executes active runtime environments (Node.js, Python, Go) that run continuously to process network sockets, execute business logic, query databases, and manage memory states.

### Real-world Analogy
Frontend hosting is like a vending machine: it stands in the hall and immediately drops pre-packaged snacks (static assets) when buttons are pressed. Backend hosting is like a busy chef working in the kitchen: they take raw orders, cook dishes, manage inventory, and boil liquids continuously.

### Code Example
```javascript
// A backend runtime server must listen on a port indefinitely
const http = require("http");

const server = http.createServer((req, res) => {
  res.end("Active runtime response");
});

server.listen(8080, () => console.log("Server listening on port 8080"));
```

### Common Interview Questions
- Why do backend services require persistent memory runtimes compared to static sites?
- Compare CPU and memory allocation strategies on backend hosts.
- What are the scaling implications of running stateful vs stateless backend services?

### Reference Links
- [AWS: What is Backend Web Development?](https://aws.amazon.com/what-is/back-end-development/)

## VPS Hosting (Virtual Private Servers)

### Definition
VPS hosting (AWS EC2, DigitalOcean Droplets) allocates isolated virtual machines with dedicated root access, operating system kernels, and firewall rules on shared physical server hardware.

### Real-world Analogy
VPS hosting is like renting a complete townhouse. You get your own lock, can paint the walls (root access), install custom pipes (install packages), and set up security gates (firewalls). You are responsible for cleaning the gutters and fixing the roof (Linux administration).

### Code Example
```bash
# Provisioning a Linux VPS server via SSH
ssh -i my-keypair.pem ubuntu@ec2-192-0-2-1.compute-1.amazonaws.com

# Simple UFW Firewall configuration
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

### Common Interview Questions
- What is SSH and why are SSH key pairs preferred over passwords for server access?
- Explain the role of a firewall (like UFW or AWS Security Groups) in VPS security.
- How do you manage system updates and package installations on a Linux VPS?

### Reference Links
- [AWS: Amazon EC2 Product Page](https://aws.amazon.com/ec2/)
- [DigitalOcean: How to Create a Droplet](https://docs.digitalocean.com/products/droplets/how-to/create/)

## Serverless Backends

### Definition
Serverless backends (AWS Lambda, Vercel Functions) execute code in response to events, automatically scaling resources up and down, charging only for active runtime milliseconds, and tearing down idle container instances.

### Real-world Analogy
Instead of hiring a chef to stand in your kitchen 24/7 paying them hourly even when no customers arrive (VPS hosting), you hire a freelance caterer who only enters the kitchen, cooks a single meal when an order slip drops (event), washes their hands, and leaves.

### Code Example
```javascript
// AWS Lambda Handler Template (Stateless execution)
exports.handler = async (event) => {
  const body = JSON.parse(event.body || "{}");
  return {
    statusCode: 200,
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ message: "Processed " + body.name }),
  };
};
```

### Common Interview Questions
- What is a serverless "Cold Start" and what factors influence its duration?
- Why do serverless functions pose a risk of database connection pool exhaustion?
- Explain the timeout and memory limitation constraints of AWS Lambda.

### Reference Links
- [AWS: Serverless Computing](https://aws.amazon.com/serverless/)
- [Vercel Docs: Serverless Functions](https://vercel.com/docs/concepts/functions/serverless-functions)

## Platform as a Service (PaaS)

### Definition
Platform as a Service (PaaS) platforms (Render, Railway) automate backend deployments by handling OS maintenance, firewalls, and server provisioning, allowing developers to deploy code directly from Git repositories.

### Real-world Analogy
PaaS is like staying at a full-service hotel. You don't build the bed, clean the bathroom, or hook up the TV cables (infrastructure). You check in, drop your bags (code), and live in the room. The hotel management handles cleaning and security.

### Code Example
```json
// package.json script configuration for PaaS deployments
// PaaS monitors package.json and automatically runs npm start on port PORT
{
  "scripts": {
    "start": "node server.js"
  }
}
```

### Common Interview Questions
- Compare the developer experience of PaaS (Render) vs VPS (AWS EC2).
- How do PaaS systems bind host environment variables (PORT) to your application runtime?
- What are the scaling limitations of PaaS platforms during high traffic?

### Reference Links
- [Render Documentation](https://docs.render.com/)
- [Railway Documentation](https://docs.railway.app/)

## Reverse Proxy with Nginx

### Definition
Nginx is a high-performance web server that acts as a Reverse Proxy, forwarding incoming HTTP/HTTPS requests to internal backend application servers (Node.js running on local ports) and managing SSL certificates, gzip compression, and caching.

### Real-world Analogy
Imagine a corporate headquarters building. Visitors don't walk directly to a clerk's private desk on floor 4 (Node.js port 3000). They talk to the receptionist in the main lobby (Nginx port 80/443). The receptionist checks their pass (SSL), translates their query, and directs them to the desk.

### Code Example
```nginx
# /etc/nginx/sites-available/default configuration snippet
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:3000; # Forward requests to Node.js
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Common Interview Questions
- Why is it bad practice to expose your Node.js application port directly to the public web?
- Explain the role of Host, X-Real-IP, and X-Forwarded-For headers in reverse proxies.
- How do you configure Nginx to serve static files directly, bypassing the backend server?

### Reference Links
- [Nginx Docs: Reverse Proxy Guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
- [Certbot: Nginx integration](https://certbot.eff.org/)

## Load Balancers

### Definition
Load Balancers distribute incoming network traffic across multiple backend servers to prevent single-point overload, using routing algorithms (Round-Robin, Least Connections) and verifying server health via periodic probes.

### Real-world Analogy
Imagine a busy bank branch. Instead of everyone queueing in front of Teller A, a receptionist directs customers to Teller A, Teller B, and Teller C sequentially (Round-Robin), ensuring no single teller gets overwhelmed while others stand idle.

### Code Example
```nginx
# Nginx Load Balancing upstream block configuration
upstream backend_servers {
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend_servers;
    }
}
```

### Common Interview Questions
- Describe the difference between Round-Robin and Least Connections load balancing algorithms.
- What are sticky sessions and why are they necessary when using WebSockets in clustered servers?
- How do health checks prevent load balancers from routing traffic to crashed servers?

### Reference Links
- [Nginx Docs: Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)

## HTTPS Setup for Backend

### Definition
HTTPS encrypts HTTP communication using SSL/TLS certificates. Setting up HTTPS on backends requires domain binding, installing Nginx, and automating certificate generation/renewals using Certbot and Let's Encrypt.

### Real-world Analogy
HTTP is like writing details on a postcard: anyone who handles the mail can read it. HTTPS is like placing the postcard inside a heavy locked titanium box that can only be unlocked by the recipient's secure key.

### Code Example
```bash
# Automated Let's Encrypt SSL certificate generation using Certbot
sudo apt update
sudo apt install certbot python3-certbot-nginx
# Certbot scans Nginx configuration, issues SSL, and configures redirects automatically
sudo certbot --nginx -d api.example.com
```

### Common Interview Questions
- How does the SSL/TLS cryptographic handshake work?
- Why do Let's Encrypt certificates expire every 90 days, and how do you automate renewals?
- What Nginx configuration enforces HTTP-to-HTTPS redirection?

### Reference Links
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [Certbot Instructions](https://certbot.eff.org/instructions)

## Zero-Downtime Deployments

### Definition
Zero-Downtime deployments release updates without interrupting system availability. Strategies include Blue-Green deployments (switching routing between identical environments), Rolling Updates (incremental node updates), and Canary releases.

### Real-world Analogy
Blue-Green is like having two identical theater stages side-by-side. The play runs on the Blue stage. While the play runs, actors prepare the new scenery on the Green stage (deployment). When ready, you turn off the Blue lights and turn on the Green lights instantly.

### Code Example
```
// Rolling Update Sequence:
// Host Cluster (3 nodes running v1.0)
//   -> Update Node 1 to v1.1, test health, join cluster
//   -> Update Node 2 to v1.1, test health, join cluster
//   -> Update Node 3 to v1.1, test health, join cluster
//   -> Release complete with zero user downtime.
```

### Common Interview Questions
- Contrast Blue-Green deployment and Rolling Update deployment models.
- What are Canary Deployments and how do they mitigate production risks?
- How do you manage database schema migrations during zero-downtime deployments (e.g. backward compatibility)?

### Reference Links
- [Martin Fowler: BlueGreen Deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html)

## Scenario-based Interview Questions for Backend Deployment

### Scenario 1
You run a Node.js backend on a single AWS EC2 instance. During peak traffic, you notice the server CPU hits 100%, and the API starts dropping user connections. How do you scale this service to support high traffic without database connection exhaustion?

*Expected Approach:*
1. Spawn multiple EC2 instances running the Node.js backend.
2. Place an Application Load Balancer (ALB) in front of the instances to distribute incoming HTTP request loads.
3. Configure connection pooling limits in the Node.js database configurations, and deploy a PGpool or Redis caching layer to prevent the scaled instances from overwhelming the single database database socket limit.

### Scenario 2
You want to deploy a new version of an API that changes a database table structure (renaming a user profile column). If you run the migration script and deploy the new code at the same time, active users get database errors. How do you execute this deployment with zero downtime?

*Expected Approach:*
1. Implement a Multi-Phase (Expand and Contract) Database Schema migration strategy.
2. Phase 1 (Expand): Add the new column to the database while keeping the old column active.
3. Deploy new code that writes to both columns and reads from the old column.
4. Phase 2: Run a migration script to copy old column data to the new column.
5. Deploy code updates that read and write exclusively from the new column.
6. Phase 3 (Contract): Run a migration script to safely drop the old column.
