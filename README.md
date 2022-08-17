# Monorepos with Turborepo and NPM Workspaces

## Lessons

### Lesson 01 - Installing Turborepo in an existing project

1. We need to install Turborepo in our project by running `npm install turbo -D` to install it in the root of our project as a `devDependency`.
2. Then, we need to update `.gitignore` to prevent `git` to include certain folders in version control. In this case, we're going to ignore `.turbo` and the output folders for all the applications in the monorepo, `dist/**` and `.next/**`
3. Create a `turbo.json` file and reference the Turborepo's JSON Schema
