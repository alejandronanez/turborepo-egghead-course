# Monorepos with Turborepo and NPM Workspaces

## Lessons

### Lesson 01 - Installing Turborepo in an existing project

1. We need to install Turborepo in our project by running `npm install turbo -D` to install it in the root of our project as a `devDependency`.
2. Then, we need to update `.gitignore` to prevent `git` to include certain folders in version control. In this case, we're going to ignore `.turbo` and the output folders for all the applications in the monorepo, `dist/**` and `.next/**`
3. Create a `turbo.json` file and reference the Turborepo's JSON Schema

### Lesson 02 - Create your first turborepo pipeline

1. Let's add the `pipeline` object in the `turbo.json` file.
2. Select the scripts we want Turborepo to take care of from our `package.json` file, then. For instance, if we want Turborepo to run the `dev` command, we have to create a `"dev"` key inside the `"pipeline"` object in `turbo.json`.
3. We can try this out with the Turborepo CLI, we just need to run `npx turbo run [your pipeline name]`
4. Make sure we add the `@monorepo/utils` package as a dependency in the `@monorepo/blog` and `@monorepo/dashboard` projects to avoid any dependency problems. For this we can use the command `npm install @monorepo/utils -w=@monorepo/blog -w=@monorepo/dashboard`. For more information, see this Egghead.io lesson [Install dependencies for specific packages in a monorepo](https://egghead.io/lessons/npm-install-dependencies-for-specific-packages-in-a-monorepo)
5. We migrate all the commands that we care about as Turborepo pipelines.

- [x] `dev` -> `npx turbo run dev`
- [x] `check` -> `npx turbo run test typecheck`
- [x] `test` -> `npx turbo run test`
- [x] `build` -> `npx turbo run build`
- [x] `typecheck` -> `npx turbo run typecheck`

6. Let's replace all the scripts we have in the root `package.json` with their turborepo counterparts.
