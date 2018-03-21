**Assumptions**: This tutorial assumes that the local development environment is Windows 7+ and is a traditional "locked down" corporate environment.

# Local Development Environment Pre-requisites
### Obtain administrator credentials / elevated privileges
Obtain elevated priveleges / firecall credentials for your machine.
### Install / Upgrade NodeJS  and npm
Note: This step could be skipped if NodeJS / npm are already present and meet minimum version requirements (see next section)
  1. Download and run the most current LTS NodeJS Windows Installation msi @ https://nodejs.org/en/download/
  2. Accept UELA, and continue through the installation with the default settings
  3. Enter administrator credentials when prompted during final install step
  4. Wait until completes with confirmation of successful installation.
### Validate Node.JS / npm installation and version requirements
  1. Launch Powershell (Ctrl + Win key -> type 'Powershell' -> Enter)  
  2. execute `node -v`
  3. Ensure version of node is **at least** v6.5.0 or later.  If not, re-install latest LTS distribution.
### Install the Serverless CLI
  1. Set npm proxy if on corporate network machine.  Execute the following commands (be sure to substitute $PROXY_HOST as necessary):
      * `npm config set https-proxy http://$PROXY_HOST:8080/`
      * `npm config set proxy http://$PROXY_HOST:8080/`
  2. Install serverless cli module:
      `npm install -g serverless`
  




1. Launch a shell with elevated privileges (Ctrl + Win key -> type 'Powershell' -> right click -> Run as Administrator -> enter adminstrator credentials)
 
