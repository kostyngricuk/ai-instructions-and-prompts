# Deploy React App to Netlify

Create reusable deploy script for deploying the app to **Netlify** using best practices:

1. A `netlify.toml` configuration file that:
   - Defines the build command as `npm run build`
   - Sets the publish directory to `build`
   - Includes a `[dev]` section so `npm start` works locally with Netlify Dev
   - Uses port `3000`

2. A reusable shell script `deploy.sh` that:
   - Runs `npm run build`
   - Deploys the `build` folder to Netlify with `--prod`
   - Adds a commit message with a timestamp
   - Is executable (`chmod +x deploy.sh`)

3. Add a new script in `package.json` to run the deploy process

4. Clear instructions on (add to `README.md` file):
   - Installing the Netlify CLI
   - Running the script to deploy
   - Setting up CI/CD via GitHub if I want automated deploys

Make the output production-ready, clean, and minimal.
