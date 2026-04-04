# GhostLink

GhostLink is a lightweight URL security intelligence scanner that analyzes links before users interact with them. It combines heuristic threat detection, security header inspection, domain intelligence, redirect analysis, and metadata extraction to produce a composite risk score and clear recommendations.

It is designed as a fully serverless front end plus Cloudflare Worker backend.

## Screenshots

![GhostLink UI Screenshot 1](Screenshot%202026-04-04%20180952.png)
![GhostLink UI Screenshot 2](Screenshot%202026-04-04%20181016.png)
![GhostLink UI Screenshot 3](Screenshot%202026-04-04%20181039.png)
![GhostLink UI Screenshot 4](Screenshot%202026-04-04%20181054.png)

## At a Glance

- Frontend: Vanilla HTML, CSS, and JavaScript
- Backend: Cloudflare Worker
- Output: Security score, risk level, findings, details, and recommendations
- Storage: No database, no URL persistence
- Deployment: Cloudflare Pages, GitHub Pages, Netlify, Vercel, or any static host

## What GhostLink Does

GhostLink evaluates a URL across multiple security dimensions:

- Phishing indicators such as IP-based URLs, @ symbols, suspicious keywords, excessive subdomains, and long domains
- Transport security checks including HTTPS validation, redirect behavior, and HSTS presence
- Security header checks including Content-Security-Policy, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, and Permissions-Policy
- Domain intelligence such as Punycode detection, suspicious TLDs, hostname entropy, numeric labels, and hyphenated labels
- Reputation heuristics such as long URLs, encoded characters, and high randomness in the path
- Redirect chain analysis to detect multi-hop or suspicious redirect behavior
- Page metadata, server information, and DNS-related diagnostics

The result is a score from 0 to 100 plus a human-readable risk level:

- 0 to 30: Low
- 31 to 70: Medium
- 71 to 100: High

## How the Analysis Flow Works

1. The user enters a URL in the frontend.
2. The frontend validates the input and normalizes missing protocols to HTTPS.
3. The browser sends a POST request to the Cloudflare Worker at `/analyze`.
4. The Worker runs the analysis pipeline:
   - phishing checks
   - HTTPS and SSL checks
   - security header checks
   - redirect chain checks
   - metadata extraction
   - server inspection
   - DNS inference
   - domain intelligence
5. The Worker calculates a final score and risk level.
6. The frontend renders:
   - Security Analysis Summary
   - findings grouped by category
   - detailed URL intelligence
   - safety recommendations

## User Interface Sections

GhostLink’s UI is organized into clear sections:

- Header: product name, subtitle, and short capability tags
- Input panel: URL field and Analyze button
- Security Analysis Summary: score, risk level, score bar, and overall explanation
- Security Findings: grouped issues detected by the scanner
- URL Intelligence: technical breakdown of protocol, domain, metadata, server, DNS, and domain intelligence
- Safety Tips: user-focused recommendations based on risk and findings
- Footer: attribution and page metadata

## Project Structure

```text
GhostLink/
├── index.html
├── style.css
├── script.js
├── SETUP.md
├── README.md
├── LICENSE
├── _headers
├── Screenshot 2026-04-04 180952.png
├── Screenshot 2026-04-04 181016.png
├── Screenshot 2026-04-04 181039.png
├── Screenshot 2026-04-04 181054.png
└── ghostlink/
    └── worker-production.js
```

## Frontend Behavior

The frontend is built with plain JavaScript and is intentionally simple.

Key behavior:

- Validates that the input is a URL
- Automatically adds `https://` when the protocol is missing
- Sends the request to the Worker API
- Expands the layout when analysis begins
- Scrolls the results into view so the summary is visible immediately
- Renders tooltips for technical terms on desktop and mobile

The main frontend configuration is the Worker URL constant in `script.js`.

## API Reference

### Endpoint

`POST /analyze`

### Request Headers

```http
Content-Type: application/json
```

### Request Body

```json
{
  "url": "https://example.com"
}
```

The URL must include `http://` or `https://`.

### Success Response

```json
{
  "success": true,
  "data": {
    "score": 42,
    "riskLevel": "Medium",
    "findings": [
      {
        "type": "No HTTPS",
        "category": "HTTPS & SSL",
        "description": "URL does not use HTTPS encryption",
        "points": 20
      }
    ],
    "details": {
      "url": "http://example.com",
      "components": {
        "protocol": "http:",
        "domain": "example.com",
        "port": "80",
        "pathDepth": 0,
        "pathLength": 1,
        "queryParamCount": 0,
        "hasFragment": false,
        "domainParts": 2
      },
      "metadata": null,
      "server": null,
      "dns": null,
      "domainIntel": null
    },
    "recommendations": [
      "⚠️ Be cautious with this site — it has some security concerns. Avoid entering sensitive data."
    ]
  }
}
```

### Error Responses

The Worker can return the following error codes:

- `INVALID_INPUT` - the JSON body did not contain a URL string
- `EMPTY_URL` - the URL value was empty after trimming
- `MISSING_PROTOCOL` - the URL did not start with `http://` or `https://`
- `ANALYSIS_ERROR` - the analysis pipeline failed
- `INTERNAL_ERROR` - the request could not be processed
- `NOT_FOUND` - any unsupported route

## Response Data Model

### score

The final numeric risk score. Higher values mean higher risk.

### riskLevel

One of the following values:

- Low
- Medium
- High

### findings

An array of detected issues. Each item includes:

- type
- category
- description
- points

### details

Technical supporting data for the analyzed URL:

- url: normalized input URL
- components: protocol, domain, port, path depth, path length, query count, and fragment presence
- metadata: page title, description, content type, language, charset, favicon, and form presence
- server: status code, status message, response time, server headers, redirect chain, security headers, and technology hints
- dns: domain name, resolved IP if available, and lookup timing
- domainIntel: registrable domain, TLD class, subdomain profile, entropy, and label indicators

### recommendations

Short, user-friendly guidance based on the detected issues and risk level.

## Scoring Logic

GhostLink uses heuristic scoring. Every finding has a point value, and some categories are weighted more heavily than others.

Weighted categories in the Worker:

- Phishing Risk: 1.3
- HTTPS & SSL: 1.4
- Security Headers: 1.2
- Domain Intelligence: 1.25
- Reputation: 1.0
- Redirect Chain: 1.35

The Worker then maps the final score into the risk levels listed earlier.

## Detection Rules

### Phishing Risk

- Direct IP address in the URL
- Use of `@` in the URL
- Excessive subdomains
- Suspicious keywords such as login, verify, secure, update, bank, and free
- Domain names that are unusually long

### HTTPS and SSL

- Missing HTTPS
- HTTP URLs that do not redirect to HTTPS
- Missing HSTS header

### Security Headers

- Missing Content-Security-Policy
- Missing X-Frame-Options
- Missing X-Content-Type-Options
- Missing Strict-Transport-Security
- Missing Referrer-Policy
- Missing Permissions-Policy

### Domain Intelligence

- Suspicious top-level domains
- Punycode or homograph-style domains
- Subdomain count and profile
- Hostname entropy
- Numeric labels
- Hyphenated labels

### Reputation

- Very long URLs
- Encoded characters in the URL
- High entropy in the path or query string

### Redirect Chain

- Redirect chains longer than the allowed threshold

## Local Development

### Requirements

- Git
- A browser
- Cloudflare account if you want to run the worker backend

### Run the Frontend Locally

You can open `index.html` directly in a browser, but a local server is recommended.

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

### Important Note

If you open the file directly with `file://`, browser fetch behavior may be restricted. Use a local server for the cleanest experience.

## Cloudflare Worker Setup

1. Open the Cloudflare dashboard.
2. Create a new Worker.
3. Copy the contents of `ghostlink/worker-production.js` into the Worker editor.
4. Deploy the Worker.
5. Copy the Worker URL ending in `/analyze`.
6. Update `WORKER_URL` in `script.js`.

Example:

```javascript
const WORKER_URL = 'https://your-worker.your-subdomain.workers.dev/analyze';
```

## Cloudflare Pages Deployment

1. Connect the repository to Cloudflare Pages.
2. Use no build command.
3. Use `/` as the output directory.
4. Deploy the site.

The repository already includes `_headers` for static hosting security headers.

## Alternative Deployment Options

### GitHub Pages

Host the repository as a static site from the main branch.

### Netlify

Drag and drop the project folder or connect the repository via Git.

### Vercel

Deploy as a static project without a build step.

## Troubleshooting

### The scanner says analysis failed

- Verify that the Worker URL in `script.js` is correct
- Confirm the Worker is deployed and reachable
- Check browser console errors

### I see a CORS error

Make sure the Worker returns the CORS headers expected by the frontend:

```javascript
{
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Methods': 'POST, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type'
}
```

### The page loads but no results appear

- Check that the URL includes `http://` or `https://`
- Confirm the Worker endpoint is `/analyze`
- Make sure the site is not being blocked by browser privacy settings or network restrictions

### Security headers look incomplete

- Some sites intentionally omit headers
- Some response headers are not exposed to cross-origin requests
- Redirects and CDN layers can change what headers are visible

## Privacy and Security Notes

- GhostLink does not store analyzed URLs.
- Analysis happens on demand.
- The project is designed for inspection, not persistence.
- Results are heuristic and should be treated as a security signal, not a final verdict.

## Limitations

- Heuristic scanners can produce false positives.
- Some sites block `HEAD` or restrict metadata access.
- DNS information is inferred where direct lookup is not available from the Worker environment.
- Redirect and header visibility can vary depending on the target site and CDN behavior.

## Customization

You can adapt GhostLink in several ways:

- Change branding in `index.html`
- Adjust the visual design in `style.css`
- Tune the frontend behavior in `script.js`
- Modify scoring logic and detection rules in `ghostlink/worker-production.js`

## License

See the [LICENSE](LICENSE) file for details.

---

Made with care by Adheesha
