# 19 Frontend Deployment

This guide covers frontend hosting: static vs server hosting, Vercel deployments, Netlify, AWS S3 + CloudFront configurations, CDN edge caching, and DNS/SSL configurations.

## Frontend Hosting Fundamentals

### Definition
Frontend hosting serves static assets (HTML, CSS, JS, images) directly to browsers. Unlike dynamic servers, static hosting does not require a running backend runtime process; assets are stored on storage disks and distributed via CDN Edge networks.

### Real-world Analogy
Imagine distributing newspapers. Server hosting is like having a printing press print a custom copy of the paper for each reader only when they walk up and ask. Static hosting is printing 10,000 papers, storing them in regional kiosks (CDNs), and handing copies out instantly.

### Code Example
```
// Static build output structure (e.g. out/ or dist/ directory)
dist/
  index.html
  assets/
    index-a9f4c.js
    style-b2d91.css
```

### Common Interview Questions
- Explain why static hosting is cheaper and scales better than dynamic server hosting.
- What is a CDN (Content Delivery Network) and how does it reduce latency?
- How does the browser cache static assets based on unique build hashes?

### Reference Links
- [AWS: What is a CDN?](https://aws.amazon.com/what-is/cdn/)
- [Cloudflare: Static vs Dynamic Content](https://www.cloudflare.com/learning/performance/static-vs-dynamic-content/)

## Vercel

### Definition
Vercel is a cloud platform optimized for frontend frameworks and static sites. It provides zero-config integration for Next.js, automatically deploys API routes as Serverless or Edge functions, and provisions Preview Deployments for git branches.

### Real-world Analogy
Vercel is like hiring a premium publisher for your book. You write the chapters in a digital system. Every time you finish a chapter draft (pull request), the publisher prints a temporary pocket book (Preview Deployment) and sends it to you to review before updating the final edition.

### Code Example
```json
// vercel.json configuration example
{
  "cleanUrls": true,
  "redirects": [
    { "source": "/old-path", "destination": "/new-path", "permanent": true }
  ]
}
```

### Common Interview Questions
- How do Vercel Preview Deployments improve pull request reviews?
- What are Edge Middleware functions and how do they differ from Serverless functions?
- How does Vercel optimize static pages using Incremental Static Regeneration (ISR)?

### Reference Links
- [Vercel Documentation](https://vercel.com/docs)

## Netlify

### Definition
Netlify is a web hosting and automation platform. It features Git-integrated continuous deployments, Netlify Functions (serverless code), custom Redirects/Rewrites rules, Forms management, and Split Testing.

### Real-world Analogy
Netlify is like renting a smart modular booth at a fair. It sets up your display boards automatically when you bring them in, provides customer feedback forms out of the box, and lets you show different layouts (split testing) to different crowds.

### Code Example
```toml
# netlify.toml configuration file
[build]
  publish = "dist"
  command = "npm run build"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### Common Interview Questions
- How do you configure A/B testing (split testing) using Netlify configuration files?
- What is the purpose of the _redirects file in Netlify deployments?
- How do you write serverless functions in Netlify?

### Reference Links
- [Netlify Documentation](https://docs.netlify.com/)

## AWS S3 + CloudFront

### Definition
AWS S3 stores static website assets, while CloudFront acts as a CDN distribution layer, serving cached assets from regional edge locations, providing SSL encryption via AWS Certificate Manager (ACM), and caching content.

### Real-world Analogy
S3 is a massive filing cabinet sitting in a central Seattle office (Origin). CloudFront is a network of local courier desks in cities around the world (Edge Locations). The courier desks keep photocopies of the documents on their tables so local clients don't have to wait for mail from Seattle.

### Code Example
```bash
# Upload build assets to S3 bucket and invalidate CloudFront cache
aws s3 sync ./dist s3://my-static-web-bucket --delete

# Invalidate CloudFront cache to force updates
aws cloudfront create-invalidation --distribution-id E1A2B3C4D5E6F7 --paths "/*"
```

### Common Interview Questions
- Why should you block public access to your S3 bucket and use Origin Access Control (OAC) with CloudFront?
- What is a CloudFront cache invalidation and why is it necessary after updating static site files?
- How do you configure SSL certificates in CloudFront using AWS Certificate Manager?

### Reference Links
- [AWS: Hosting Static Website on S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [AWS: CloudFront Developer Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)

## GitHub Pages

### Definition
GitHub Pages is a static site hosting service that takes HTML, CSS, and JavaScript files directly from a repository on GitHub, optional running them through a build process, and publishes them.

### Real-world Analogy
Imagine writing a document in a public library room. Instead of printing the pages and walking them to a display window, you move the file into a designated public folder on the library desk. The librarian automatically frames the sheets and mounts them on the library window.

### Code Example
```yaml
# Simple GitHub Actions workflow step to deploy to GitHub Pages
- name: Deploy to GitHub Pages
  uses: JamesIves/github-pages-deploy-action@v4
  with:
    folder: dist # Folder to publish
    branch: gh-pages # Target branch
```

### Common Interview Questions
- How do you configure custom domains with SSL on GitHub Pages?
- What are the limitations of GitHub Pages compared to professional hosts like Vercel or AWS?
- How do you bypass Jekyll processing on GitHub Pages repositories (e.g. .nojekyll)?

### Reference Links
- [GitHub Pages Documentation](https://pages.github.com/)

## CDN Edge Caching

### Definition
CDN Edge Caching stores copies of web assets on servers located close to users (Edge Nodes). It uses HTTP Cache-Control headers to determine caching duration and dynamic routing rules to manage user requests.

### Real-world Analogy
Imagine a restaurant chain. Instead of cooking every meal at the master Chicago kitchen and flying the plates to diners in Tokyo, Paris, and Sydney (high latency), the chain builds local kitchens (edge nodes) in those cities that store pre-made cakes (edge cache) to serve immediately.

### Code Example
```http
// Typical CDN Cache Control Response Header
Cache-Control: public, max-age=31536000, immutable
```

### Common Interview Questions
- Explain the difference between max-age and s-maxage directive in Cache-Control headers.
- What does stale-while-revalidate do in CDN caching configurations?
- How do CDNs manage cache eviction when edge nodes run out of memory?

### Reference Links
- [Cloudflare: What is Caching?](https://www.cloudflare.com/learning/cdn/what-is-caching/)

## Custom Domains and SSL

### Definition
Custom Domains map human-readable names (example.com) to server IP addresses using DNS records (A, CNAME, TXT). SSL/TLS protocols encrypt connection channels, using certificates generated by authorities like Let's Encrypt.

### Real-world Analogy
DNS is a telephone directory: you look up the name "Bank of America" to find their telephone number (IP address). SSL is a secure soundproof room: before you discuss cash details, you check the notary certificate hanging on the wall to verify the bankers are legitimate.

### Code Example
```
// DNS Configuration Table Example
Type   Name      Value                       TTL
A      @         192.0.2.1 (Server IP)       3600
CNAME  www       my-app.vercel.app           3600
TXT    @         verification-token-102      3600
```

### Common Interview Questions
- What is the difference between an A Record and a CNAME Record?
- How does the Let's Encrypt automated ACME protocol issue SSL certificates?
- How do you enforce HTTP-to-HTTPS redirection on web host applications?

### Reference Links
- [Cloudflare: What is DNS?](https://www.cloudflare.com/learning/dns/what-is-dns/)
- [Let's Encrypt: How it Works](https://letsencrypt.org/how-it-works/)

## SPA Routing Fix

### Definition
The Single Page Application (SPA) routing fix resolves 404 errors on page refreshes. Because client-side routing (React Router) updates URLs in the browser without server calls, server configurations must rewrite all non-file requests back to index.html.

### Real-world Analogy
Imagine a house with 5 bedrooms, but only one main front door. If a guest flies in on a helicopter and tries to land directly on the window sill of bedroom 3 (refreshing /profile route), they fall. You must place a slide at every window that redirects arriving guests back to the main front door (index.html), where the host guides them to the room.

### Code Example
```json
// vercel.json SPA rewrite config (forces all routing requests to index.html)
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

### Common Interview Questions
- Why do React/Vue/Vite projects return 404 Not Found errors on page refreshes when using client-side routing?
- How do you configure AWS CloudFront error pages to fix SPA routing issues?
- What redirection rule configuration fixes SPA routing on Netlify?

### Reference Links
- [Vite Docs: Backend Integration SPA routing](https://vitejs.dev/guide/backend-integration.html)

## Scenario-based Interview Questions for Frontend Deployment

### Scenario 1
You deployed a React application to an AWS S3 bucket distributed via CloudFront. Users report that when they navigate the app using buttons, everything works. However, if they refresh the page at /dashboard, they get an S3 XML "Access Denied" or 404 error. How do you resolve this?

*Expected Approach:*
1. Explain that since it is a Single Page Application (SPA), /dashboard is a client-side route. When refreshed, the browser asks S3 for a physical file named "dashboard", which does not exist, triggering a 404.
2. Fix this by configuring CloudFront Error Pages: create a custom error response rule for 403 and 404 errors.
3. Set the response path to /index.html and return status code 200, letting React Router intercept the URL and render the dashboard view.

### Scenario 2
You just pushed a critical hotfix to your production website hosted on Vercel/Netlify. However, customers complain they are still seeing the old version of the landing page. Why is this happening and how do you diagnose and fix it?

*Expected Approach:*
1. Explain that the browser or CDN Edge Nodes have cached the old index.html file based on active Cache-Control headers (e.g. long max-age settings).
2. If assets use unique content hashes in filenames (like index-a9f4c.js), they are safe. However, index.html itself must be set to Cache-Control: no-cache or must be invalidated.
3. Trigger a CDN cache purge (CloudFront invalidation or Vercel deployment redeploy) and instruct users to perform a hard refresh (Ctrl+F5) to clear their browser cache.
