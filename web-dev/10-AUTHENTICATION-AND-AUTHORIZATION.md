# 10 Authentication and Authorization

This guide covers web security, authentication mechanisms, token rotations, secure cookie parameters, OAuth flows, and OWASP threat prevention.

## Authentication vs Authorization

### Definition
Authentication is the process of verifying who a user is (credentials verification). Authorization is the process of verifying what specific resources or actions a user is permitted to access (permissions checking).

### Real-world Analogy
Imagine walking into a secure corporate office. Checking in at the reception desk, showing your driver's license, and getting a badge is authentication. Using that badge to open the server room door (which fails if you are a visitor but succeeds if you are the network admin) is authorization.

### Code Example
```javascript
// Simple route demonstrating both concepts
app.post("/login", authenticateUser); // Authentication

app.get("/admin/panel", authorizeRole("admin"), (req, res) => {
  res.send("Welcome Admin"); // Authorization
});
```

### Common Interview Questions
- Explain the key differences between authentication and authorization.
- Give an example of a security bug that is an authorization failure rather than an authentication failure.
- How do HTTP status codes 401 Unauthorized and 403 Forbidden map to these concepts?

### Reference Links
- [Okta: Authentication vs Authorization](https://www.okta.com/identity-101/authentication-vs-authorization/)
- [MDN Web Docs: HTTP Authorization](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization)

## Session-based Authentication

### Definition
Session-based authentication stores a session identifier in the user's browser cookie while keeping the session data (user details, status) on the server in memory or in a database store like Redis.

### Real-world Analogy
Think of checking your coat at a theater. The attendant takes your coat and hands you a small plastic token with a number (session ID cookie). When you want your coat back, you show the token. The attendant looks at the number, finds your coat on the rack (server-side store), and returns it.

### Code Example
```javascript
const session = require("express-session");
const RedisStore = require("connect-redis").default;
const { createClient } = require("redis");

const redisClient = createClient();
redisClient.connect().catch(console.error);

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: "session_secret_key",
  resave: false,
  saveUninitialized: false,
  cookie: { secure: true, httpOnly: true, sameSite: "strict" }
}));
```

### Common Interview Questions
- How does session-based authentication handle horizontal scaling across multiple servers?
- Why is Redis preferred over local server memory for session storage?
- Describe the vulnerability risks of session fixation and session hijacking.

### Reference Links
- [Express Session Documentation](https://github.com/expressjs/session)
- [OWASP: Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)

## Token-based Authentication with JWT

### Definition
JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties. The token is composed of three parts (Header, Payload, Signature) separated by dots, and is cryptographically signed but not encrypted by default.

### Real-world Analogy
Think of a physical movie ticket. The front of the ticket prints your seat number and the movie title (payload). The ticket is stamped with a unique hologram seal (signature) by the theater printer. The ticket collector does not call the box office database to check your identity; they verify the hologram seal is real.

### Code Example
```javascript
const jwt = require("jsonwebtoken");

const payload = { userId: "123", role: "admin" };
const secret = "secret_key";

// Create token
const token = jwt.sign(payload, secret, { expiresIn: "15m" });

// Verify token
const decoded = jwt.verify(token, secret);
console.log(decoded.userId);
```

### Common Interview Questions
- Why is it dangerous to store sensitive data like passwords or credit cards in a JWT payload?
- Explain the role of the header, payload, and signature components in a JWT.
- What is the difference between token encoding (Base64) and token encryption?

### Reference Links
- [JWT Introduction](https://jwt.io/introduction)
- [RFC 7519: JSON Web Token Specification](https://datatracker.ietf.org/doc/html/rfc7519)

## Access Tokens and Refresh Tokens

### Definition
Access tokens are short-lived tokens (e.g. 15 minutes) used to authenticate API requests. Refresh tokens are long-lived tokens (e.g. 7 days) stored securely and used to request new access tokens when they expire. Refresh Token Rotation invalidates older tokens on use to prevent theft.

### Real-world Analogy
Imagine staying at a luxury resort. You receive a digital keycard (access token) that unlocks your room door. For safety, the card expires every 2 hours. If you want to reactivate the card, you go to the concierge desk and show your VIP membership passport booklet (refresh token) to receive a fresh keycard.

### Code Example
```javascript
// Endpoint to refresh access token
app.post("/token/refresh", async (req, res) => {
  const { refreshToken } = req.body;
  if (!refreshToken) return res.sendStatus(401);
  
  // Verify token and check database whitelist/blacklist
  jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);
    const newAccessToken = jwt.sign({ id: user.id }, process.env.ACCESS_TOKEN_SECRET, { expiresIn: "15m" });
    res.json({ accessToken: newAccessToken });
  });
});
```

### Common Interview Questions
- Why do we separate auth credentials into an access token and a refresh token?
- Explain how Refresh Token Rotation protects against token theft.
- How do you revoke a user's session if you use stateless JWT access tokens?

### Reference Links
- [Auth0: Refresh Tokens](https://auth0.com/docs/secure/tokens/refresh-tokens)
- [OWASP: JSON Web Token Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)

## Secure Cookie Configuration

### Definition
Secure cookie configurations control cookie access via HTTP flags: httpOnly (prevents client-side script reads, protecting against XSS), Secure (requires HTTPS delivery), and sameSite (controls cross-site requests, protecting against CSRF).

### Real-world Analogy
Think of a bank safety deposit box. The bank manager puts a security shield around the box (httpOnly). You cannot inspect the box using a handheld camera (JavaScript document.cookie); the box is only opened when you stand physically in the secure vault (HTTPS transmission) with the banker.

### Code Example
```javascript
res.cookie("token", token, {
  httpOnly: true, // Safeguards against XSS reads
  secure: true,   // Transmit only over HTTPS
  sameSite: "strict", // Strict blocks all cross-site cookie transfers
  maxAge: 1000 * 60 * 60 * 24 // 1 day
});
```

### Common Interview Questions
- Why does the httpOnly flag prevent Cross-Site Scripting (XSS) from reading your auth token?
- Compare sameSite: 'strict', sameSite: 'lax', and sameSite: 'none'.
- Why is it recommended to set cookie paths and domain constraints?

### Reference Links
- [MDN Web Docs: HTTP Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [OWASP: Cookie Database Security](https://cheatsheetseries.owasp.org/cheatsheets/HttpOnly_Cheat_Sheet.html)

## OAuth 2.0 Fundamentals

### Definition
OAuth 2.0 is an industry-standard authorization framework. It defines roles (Resource Owner, Client, Authorization Server, Resource Server) and flows (Authorization Code Flow with PKCE for single-page/mobile apps, Client Credentials for backend-to-backend communication).

### Real-world Analogy
Imagine staying at a hotel. Instead of giving the front desk clerk your home house key (password) so they can enter and pick up your bag, you log into your home home security app and authorize a temporary, limited door passcode (OAuth access token) that works only for their bag retrieval.

### Code Example
```javascript
// Authorization code request URL structure (Authorization Server redirect)
const oauthUrl = `https://github.com/login/oauth/authorize?client_id=${CLIENT_ID}&redirect_uri=${REDIRECT_URI}&scope=user`;
```

### Common Interview Questions
- Explain the step-by-step handshake in the OAuth 2.0 Authorization Code Flow.
- Why is PKCE (Proof Key for Code Exchange) required for single-page applications?
- What is the difference between OAuth 2.0 and OpenID Connect (OIDC)?

### Reference Links
- [OAuth.net 2.0 Specification](https://oauth.net/2/)
- [IETF: OAuth 2.0 Security Best Practices](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics)

## Social Login Implementation

### Definition
Social login allows users to sign in using their existing identity profiles from platforms like Google or GitHub. It uses libraries like Passport.js to configure strategy handshakes and map profile fields to database users.

### Real-world Analogy
Instead of registering a custom member account and filling out a detailed ID form at a new library, you show your government passport card. The librarian checks the passport seal, copies your name, and grants entry.

### Code Example
```javascript
const passport = require("passport");
const GoogleStrategy = require("passport-google-oauth20").Strategy;

passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: "/auth/google/callback"
  },
  async (accessToken, refreshToken, profile, done) => {
    // Locate or create user in database
    const user = await findOrCreateUser(profile.id, profile.emails[0].value);
    return done(null, user);
  }
));
```

### Common Interview Questions
- How do you link multiple social accounts (e.g. Google and GitHub) to a single user profile in your database?
- How do you handle authentication callbacks safely on your backend server?
- Why do you need to register redirect whitelist domains in your Google/GitHub developer consoles?

### Reference Links
- [Passport.js Documentation](https://www.passportjs.org/)

## NextAuth.js / Auth.js

### Definition
NextAuth.js (Auth.js) is a complete open-source authentication library designed for Next.js. It manages providers, callbacks, secure sessions, and supports both stateless JWT strategies and database-backed session strategies.

### Real-world Analogy
NextAuth is like hiring a premium, pre-configured building security team that comes with security badge printers, lobby check desks, social ID readers, and card swipe templates designed specifically to fit your brand-new office layouts.

### Code Example
```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from "next-auth";
import GithubProvider from "next-auth/providers/github";

const handler = NextAuth({
  providers: [
    GithubProvider({
      clientId: process.env.GITHUB_ID!,
      clientSecret: process.env.GITHUB_SECRET!,
    }),
  ],
  callbacks: {
    async session({ session, token }) {
      session.user.id = token.sub!;
      return session;
    }
  }
});

export { handler as GET, handler as POST };
```

### Common Interview Questions
- Compare the JWT session strategy and the Database session strategy in NextAuth.
- What do the NextAuth callback hooks (e.g. jwt, session) allow you to do?
- How do you protect API and page routes using NextAuth middleware?

### Reference Links
- [NextAuth.js Documentation](https://next-auth.js.org/)

## Password Security

### Definition
Password security involves protecting user credentials at rest. Passwords must never be stored in plain text. Instead, they are hashed using one-way algorithms like bcrypt, which add a unique random salt and perform computationally expensive iteration rounds to block rainbow-table and brute-force cracking.

### Real-world Analogy
Password hashing is like running a key through a wood chipper: you get a pile of unique wood chips (hash). You cannot re-assemble the wood chips back into the key (one-way). When someone brings their key, you run it through the same chipper and compare the chip pile.

### Code Example
```javascript
const bcrypt = require("bcrypt");

async function registerUser(password) {
  const saltRounds = 12; // Computation complexity level
  const passwordHash = await bcrypt.hash(password, saltRounds);
  return passwordHash;
}

async function loginUser(inputPassword, storedHash) {
  const isMatch = await bcrypt.compare(inputPassword, storedHash);
  return isMatch;
}
```

### Common Interview Questions
- Why are MD5 and SHA-256 not recommended for password hashing in web applications?
- What is a salt in cryptographic hashing and how does it prevent rainbow table attacks?
- What determines the optimal number of salt rounds when using bcrypt?

### Reference Links
- [OWASP: Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)

## Password Reset Flow

### Definition
The password reset flow allows users to recover accounts. It generates a short-lived, single-use cryptographically secure random token, stores its hashed representation in the database with an expiration date, and emails the reset link to the user.

### Real-world Analogy
Imagine losing your safety deposit keycard. The bank mails a secure voucher to your home address that is only valid for 1 hour. You bring the voucher to the counter. The banker verifies it, lets you open the box to set a new keycard, and tears up the voucher.

### Code Example
```javascript
const crypto = require("crypto");

function generateResetToken() {
  const resetToken = crypto.randomBytes(32).toString("hex");
  const hashedToken = crypto.createHash("sha256").update(resetToken).digest("hex");
  const tokenExpiry = Date.now() + 3600000; // 1 Hour
  
  return { resetToken, hashedToken, tokenExpiry };
}
```

### Common Interview Questions
- Why should you store the hashed reset token in the database rather than the plain token?
- How do you prevent username enumeration attacks on forgot-password endpoints?
- Why must reset tokens be invalidated immediately after their first use?

### Reference Links
- [OWASP: Forgot Password Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)

## Email Verification Flow

### Definition
Email verification validates ownership of user signups. The server generates a unique verification token on signup, restricts the account's access permissions, and marks the email status as verified in the database only when the verification link is clicked.

### Real-world Analogy
Think of ordering a credit card. The bank sends the card to your billing address, but it remains locked. You must call from your personal phone number (verification action) to activate the card before you can use it to pay for goods.

### Code Example
```javascript
// Middleware checking verification status
const checkEmailVerified = (req, res, next) => {
  if (!req.user.isEmailVerified) {
    return res.status(403).json({ error: "Please verify your email first" });
  }
  next();
};
```

### Common Interview Questions
- How do you block unverified accounts from accessing protected API endpoints?
- What is the best strategy to handle expired email verification tokens?
- How do you prevent signup spam loops on verification endpoints?

### Reference Links
- [OWASP: User Registration Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/User_Registration_Cheat_Sheet.html)

## Role-based Access Control (RBAC)

### Definition
Role-Based Access Control (RBAC) restricts system access to authorized users based on their assigned role profiles (e.g. Admin, Editor, Guest) and associates specific permissions with each role.

### Real-world Analogy
Think of an amusement park pass. A General Admission pass lets you enter the park and ride basic attractions. A VIP pass lets you enter the park, ride attractions, and enter the private VIP restaurant. A worker pass grants access to the mechanical control room.

### Code Example
```javascript
const checkPermission = (requiredPermission) => {
  return (req, res, next) => {
    const userPermissions = req.user.permissions; // e.g. ['read:posts', 'delete:posts']
    if (!userPermissions.includes(requiredPermission)) {
      return res.status(403).json({ error: "Access Denied" });
    }
    next();
  };
};

app.delete("/posts/:id", authenticateToken, checkPermission("delete:posts"), (req, res) => {
  res.send("Deleted");
});
```

### Common Interview Questions
- Compare Role-Based Access Control (RBAC) and Attribute-Based Access Control (ABAC).
- How do you implement resource-level authorization (e.g. User A can only edit their *own* posts)?
- How do you design a relational database schema to support many-to-many roles and permissions?

### Reference Links
- [NIST: Role-Based Access Control](https://csrc.nist.gov/projects/role-based-access-control)

## Two-Factor Authentication (2FA)

### Definition
Two-Factor Authentication (2FA) adds a security layer by requiring two verification items: something you know (password) and something you have (TOTP - Time-Based One-Time Password key synced on an authenticator app via otpauth URIs).

### Real-world Analogy
Think of opening a secure safety deposit box. You need your personal key (password: something you know) AND the bank manager must insert the physical master key at the same time (authenticator token: something you have).

### Code Example
```javascript
const otplib = require("otplib");
const qrcode = require("qrcode");

const secret = otplib.authenticator.generateSecret();
// Generate otpauth uri for Google Authenticator apps
const otpauth = otplib.authenticator.keyuri("user@test.com", "MyApp", secret);

qrcode.toDataURL(otpauth, (err, imageUrl) => {
  // Client scans this QR code image to sync their app
  console.log(imageUrl);
});
```

### Common Interview Questions
- How does the Time-Based One-Time Password (TOTP) algorithm compute a code without contacting the server?
- Why are SMS-based 2FA tokens considered less secure than authenticator apps (e.g. SIM swapping)?
- What are backup recovery codes and how should they be generated/stored?

### Reference Links
- [RFC 6238: TOTP Specification](https://datatracker.ietf.org/doc/html/rfc6238)
- [otplib GitHub Repository](https://github.com/yeojz/otplib)

## API Keys

### Definition
API Keys are unique tokens used to identify and authenticate requests from external client integrations rather than individual end-users. They are generated using secure random bytes and rate-limited per key.

### Real-world Analogy
Think of a warehouse keycard issued to a delivery company. It doesn't identify the specific driver; it authorizes the delivery company's truck to back up to the loading dock and drop off packages.

### Code Example
```javascript
const crypto = require("crypto");

function generateApiKey() {
  // Generate a long, secure API key
  const rawKey = crypto.randomBytes(32).toString("base64");
  const hashedKey = crypto.createHash("sha256").update(rawKey).digest("hex");
  return { rawKey, hashedKey };
}
```

### Common Interview Questions
- Why should API keys be stored hashed in the database while the client holds the plain key?
- How do you implement custom rate limiting per API key?
- What is the best strategy to facilitate API key rotation without causing client downtime?

### Reference Links
- [OWASP: API Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/API_Security_Cheat_Sheet.html)

## Security Headers

### Definition
Security headers are HTTP response parameters configured by middleware (like Helmet.js) to instruct the browser on how to handle content execution, frame rendering, and data transmission restrictions.

### Real-world Analogy
Imagine a safety handbook attached to a cargo package. The handbook instructs the courier: "Wear gloves when opening (X-Content-Type-Options), do not open this inside another box (X-Frame-Options), and only read instructions over secure radios (Strict-Transport-Security)".

### Code Example
```javascript
// Configuring security headers using Helmet
const helmet = require("helmet");

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "https://apis.google.com"]
    }
  }
}));
```

### Common Interview Questions
- What is the Content Security Policy (CSP) header and how does it prevent malicious script injection?
- How does the X-Frame-Options header protect against Clickjacking attacks?
- What does the Strict-Transport-Security (HSTS) header enforce?

### Reference Links
- [MDN Web Docs: HTTP Headers Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [Helmet.js Documentation](https://helmetjs.github.io/)

## CORS Deep Dive

### Definition
Cross-Origin Resource Sharing (CORS) is a security protocol that overrides default Same-Origin Policy blocks. Browsers use headers to verify if cross-origin resources are allowed, dispatching preflight OPTIONS requests before modifications.

### Real-world Analogy
Imagine an ambassador trying to cross a border. Instead of letting the diplomat pass with their baggage immediately, the border guards make a phone call to the capital (OPTIONS request) to verify the visa rules. Once confirmed, they allow the diplomat to enter.

### Code Example
```javascript
const cors = require("cors");

const corsOptions = {
  origin: "https://myfrontend.com",
  methods: "GET,POST,PUT,DELETE",
  allowedHeaders: "Content-Type,Authorization",
  credentials: true // Allow cookies/authorization headers
};

app.use(cors(corsOptions));
```

### Common Interview Questions
- Why does setting Access-Control-Allow-Origin: '*' fail when Access-Control-Allow-Credentials is set to true?
- What triggers an HTTP preflight OPTIONS request?
- How do you configure CORS to support multiple trusted origins dynamically?

### Reference Links
- [MDN Web Docs: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

## CSRF Prevention

### Definition
Cross-Site Request Forgery (CSRF) forces users to execute unwanted actions on web apps where they are currently authenticated. Prevention includes using sameSite cookies, CSRF validation tokens, and verifying Origin headers.

### Real-world Analogy
Imagine a trickster tricking a mail carrier into dropping a forged check into a bank mailbox. If the bank checks the sign-off signature slip (CSRF token) that only the user possesses and verifies the mailing address, they identify the trick and destroy the check.

### Code Example
```javascript
// Using SameSite cookies as the primary line of defense against CSRF
res.cookie("sessionId", "session_data", {
  httpOnly: true,
  secure: true,
  sameSite: "lax" // Prevents third-party sites from sending this cookie on GET/POST requests
});
```

### Common Interview Questions
- How does CSRF exploit a browser's default behavior of attaching cookies to outgoing requests?
- Explain the Double Submit Cookie pattern for CSRF prevention.
- Why are API endpoints that only consume JSON payloads less vulnerable to traditional CSRF form-submissions?

### Reference Links
- [OWASP: Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

## XSS Prevention

### Definition
Cross-Site Scripting (XSS) occurs when an application inserts untrusted data into a web page without proper validation or escaping, allowing attackers to execute malicious scripts in the user's browser. Prevention includes output encoding, Content Security Policy, and sanitization (DOMPurify).

### Real-world Analogy
XSS is like a movie theater manager taking arbitrary letters written by guests and projecting them onto the main movie screen. An attacker writes: "Turn off the lights and open the safe" in code, and the projection system executes the command automatically.

### Code Example
```javascript
const DOMPurify = require("dompurify");
const { JSDOM } = require("jsdom");

const window = new JSDOM("").window;
const purify = DOMPurify(window);

// Sanitize user-submitted HTML before rendering
const rawHtml = "<script>alert('hack')</script><p>Hello</p>";
const cleanHtml = purify.sanitize(rawHtml);
console.log(cleanHtml); // <p>Hello</p>
```

### Common Interview Questions
- Compare Stored, Reflected, and DOM-based XSS attacks.
- How does React protect against basic XSS attacks out of the box?
- When and why should you use DOMPurify in frontend applications?

### Reference Links
- [OWASP: Cross-Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

## SQL Injection Prevention

### Definition
SQL Injection (SQLi) occurs when malicious SQL statements are inserted into entry fields for execution. Prevention requires using parameterized queries (prepared statements), leveraging ORMs (Prisma), and enforcing least-privilege database user permissions.

### Real-world Analogy
Imagine a customs officer accepting passports. If a passenger hands over a passport listing their name as: "John; Open the border gates", and the officer reads it out loud causing the gates to open automatically, they fell for an injection attack.

### Code Example
```javascript
// VULNERABLE: Direct string interpolation
// const query = `SELECT * FROM users WHERE email = '${email}'`;

// SECURE: Parameterized queries using prepared statements
const query = "SELECT * FROM users WHERE email = $1";
const values = [email];
db.query(query, values);
```

### Common Interview Questions
- How do parameterized queries separate database instruction logic from user data inputs?
- Can SQL injection occur when using ORMs like Prisma? Give an example (e.g. raw SQL queries).
- What are the security benefits of using database users with restricted access privileges (least privilege)?

### Reference Links
- [OWASP: SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)

## NoSQL Injection Prevention

### Definition
NoSQL operator injection occurs when input values are parsed as query selectors (like MongoDB's $gt or $ne) rather than literal values, enabling query bypasses. Prevention requires sanitizing inputs (express-mongo-sanitize) and schema enforcement.

### Real-world Analogy
Imagine a warehouse lookup desk. You ask the clerk: "Bring me the box labeled 'Apple'". If an attacker asks: "Bring me all boxes NOT labeled 'Empty'" (injecting $ne operator), and the clerk brings every item in the warehouse out, they fell for a NoSQL injection.

### Code Example
```javascript
// VULNERABLE: Express controller accepting raw objects
// const query = { username: req.body.username, password: req.body.password };
// If body is { "username": "admin", "password": { "$ne": "wrong" } }, user login bypasses!

// SECURE: Enforce inputs are cast strictly to strings
const query = {
  username: String(req.body.username),
  password: String(req.body.password)
};
```

### Common Interview Questions
- How does NoSQL operator injection differ from relational SQL injection?
- What does express-mongo-sanitize do to request payloads?
- How do Mongoose schemas help prevent query selector injection?

### Reference Links
- [OWASP: Testing for NoSQL Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.6-Testing_for_NoSQL_Injection)

## Scenario-based Interview Questions for Authentication & Security

### Scenario 1
Your product team wants to store JWT access tokens in browser LocalStorage because it is simple to implement and share across client pages. As the senior developer, why do you object to this decision and what is your recommended approach?

*Expected Approach:*
1. Object to LocalStorage because it is accessible to any JavaScript code running on the same domain, making the token highly vulnerable to theft via Cross-Site Scripting (XSS) attacks.
2. Recommend storing JWT access tokens in memory (React state) and short-lived session cookies configured with the httpOnly, Secure, and sameSite: "lax" or "strict" flags.
3. This completely blocks JavaScript reads while retaining automatic, secure token delivery on backend requests.

### Scenario 2
You noticed an unusual traffic surge on your application's sign-up endpoint. Attackers are running automated bots to register fake user emails, filling your database with junk records and exhausting SMTP verification email limits. How do you mitigate this?

*Expected Approach:*
1. Implement route-level rate limiting on the signup endpoint based on client IP addresses (e.g. maximum of 5 signups per hour).
2. Integrate a CAPTCHA system (like Cloudflare Turnstile or Google reCAPTCHA v3) to detect and block non-human form submissions.
3. Add domain validation to reject signups from known temporary email generators and enforce strict input validation matching format criteria.
