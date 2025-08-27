# Deploy the App to Netlify

Create reusable deploy script for deploying the app to **Netlify** using best practices:

## Tasks

[ ] 1.0 A `netlify.toml` configuration file that:
  [ ] 1.1 Defines the deploy command
  [ ] 1.2 Sets the publish build directory

[ ] 2.0 A reusable shell script `deploy.sh` that:
  [ ] 2.1 Runs the deploy command
  [ ] 2.2 Deploys the build directory to Netlify with `--prod`
  [ ] 2.3 Adds a commit message with a timestamp
  [ ] 2.4 Is executable (`chmod +x deploy.sh`)

[ ] 3.0 Add a new script in the `package.json` to run the deploy process

[ ] 4.0 Clear instructions on (add to `README.md` file):
  [ ] 4.1 Installing the Netlify CLI
  [ ] 4.2 Running the script to deploy