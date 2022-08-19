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

### Lesson 03 - Pipeline dependencies

In some cases, you want to make sure that Turborepo is building the dependencies in a specific order, for example, we always want to build the `@monorepo/utils` first, before building the `@monorepo/dashboard` and `@monorepo/blog` projects. For that, we have to add the `"dependsOn"` key to the pipeline we want to behave like that.

```diff
  "build": {
+   "dependsOn": ["^build"],
    "outputs": [".next/**", "dist/**"]
  },
```

In some other cases where you need more granular control over what to build first in different projects you can do the following:

#### 🚨This is a contrived example🚨

Turborepo is already taking care of building things in the right order because `@monorep/utils` is a dependency of `@monorepo/dashboard` and `@monorepo/blog` and that we're already specifying that `@monorepo/dashboard` and `@monorepo/blog` depend on the build step from `@monorepo/utils` to succeed. All that said, for this lesson we're going to remove the `@monorepo/utils` dependencies from `@monorepo/dashboard` and `@monorepo/blog`, then we delete the build artifacts from `@monorepo/utils` (the `dist` folder), and finally we'll run `npm install`. Once that's done we're going to run `npm run build` and we'll see that we have a problem with our build because we're building things "out of sync", that means that there's no way for Turborepo to tell which project to build first since we don't specify that `@monorepo/utils` is a dependency of `@monorepo/blog` and `@monorepo/dashboard`.
The way to fix this would be to define a pipeline for both projects `@monorepo/dashboard` and `@monorepo/blog`.

With this we're telling Turborepo to always run the `@monorepo/utils` project, before running the build command in `@monorepo/dashboard` and `@monorepo/blog`

In the `package.json` file of `@monorepo/dashboard` and `@monorepo/blog`

#### Note

All the patch for this lesson can be found in the `/patches/lesson-03.patch` file.

```diff
  "dependencies": {
-   "@monorepo/utils": "^1.0.0",
```

In `turbo.json`

```diff
 "pipeline": {
+   "@monorepo/dashboard#build": {
+     "dependsOn": ["@monorepo/utils#build"]
+   },
+   "@monorepo/blog#build": {
+     "dependsOn": ["@monorepo/utils#build"]
+   },
```

### Lesson 04 - The dependency graph

For this we need to install [Graphviz](https://graphviz.org/download/) first. Once we have that installed, we can run `npx turbo run [pipeline to inspect] --graph=name-of-the-graph.{svg|jpg|json|pdf|png|html}` and it will generate the file you specified for you. I found this to be useful to understand how your projects depend on each other.

![](images/build-deps.jpg)

### Lesson 05 - The filter command

We already have a few npm scripts that are not using Turborepo, instead they are using _raw_ npm scripts, like `npm run typecheck -w @monorepo/dashboard`.

In order to tell Turborepo to run just in certain projects, we need to use the `--filter` flag.

[Official `--filter` documentation](https://turborepo.org/docs/core-concepts/filtering#filter-by-changed-packages)

To test out the filters we will use the patch that's in `patches/lesson-05-filter-command.patch`, lets:

1. `git checkout -b lesson-5` You can name your branch whatever you want.
2. `git apply patches/lesson-05-filter-command.patch`
3. Start filtering out

- [ ] By package
- [ ] By dependants
- [ ] By dependencies
- [ ] Exclude
- [ ] Git changed packages

Now, let's update the npm scripts we have in our root `package.json` file

### Lesson 06 - The `continue` command

Turborepo will terminate the process it's running if it encounters an error, we can prevent this from happening if we pass the `--continue=true` flag to the Turborepo command we want to execute.

```shell
npx turbo run test --continue=true
```

1. Make the tests in `utils/` to fail
2. Run `npx turbo run test`
3. See what happens
4. Add the `--continue=true` flag to the `npx turbo run test` command

I have found this to be useful when I want to know the impact of my changes across the whole project. Given the fact that `@monorepo/utils` is a dependency of `@monorepo/dashboard` and `@monorepo/blog` it's important to know the impact of my changes not only in the `@monorepo/utils` package, but in the packages that use it.
