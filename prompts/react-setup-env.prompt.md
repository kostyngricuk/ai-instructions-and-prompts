# Setup Environment

Main rule: it should be very simple app - base boilerplate for reusing in any future project

- Install React and TypeScript (Redux Toolkit)
- Use `npm` instead of `yarn`
- Install and configure bundler (Vite) - stc, test and prod envs
- Setup proxy and envs to make API requests on (stg, test and prod envs)
- Use .env file for all general and secure variables (keep only .env.template - other should be in .gitignore)
- Setup linting (ESLint) and TS checks
- Install and configure `styled-components` and UI library (MUI) - add default theme for better customization
- Add GitLab Pipeline for running Lint and TS checks for each created PR
- Create `README.md` file with detail information about the environment and local running
- follow React best practices follow by article recommendations https://medium.com/front-end-weekly/top-react-best-practices-in-2025-a06cb92def81
