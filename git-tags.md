## Versioning with git tags for git-module 

### Use case
I'd like to use a dependency module as a git module inside of my applications for quicker development. 
This would replace the need to publish to npm as a user would just need to `git clone` the dep module into
the application (s)he is working on. This would also allow for the ability to try out features on other development branches with ease,
develop and test much quicker without having to stage, commit, push, MR, wait, publish, and pull for each change to test in the main application, and
 this would also enforce common linting and defined rules across both applications.
 
 ### The problems
 1. Changes are made to the shared module that break the others unexpectedly
 2. It's more difficult to check out an older release and know that the code is the same (e.g. a version tag on enum)
 3. Increased compared to static assets? (this needs to be tested)
 4. Linter, bundlers, and resolvers need to be carefully configured to not interfere with the sub-module
 
 ### Solutions
 1. Changes made to the shared module need to be done in a branch, like all other repos and 
  follow the process of creating MRs that everyone would approve. Like any other repo, it would go through
  a process which _helps_ reduce the chance of breaks on host applications. This alongside proper git version management should
  allow for the flexibity and ensurance that typical npm versioning gives us.
  
 2. xxx help here xxx
 
 3. This needs to be tested 
 
 4. This is a one-time setup that can be documented
 
 ### npm version
 `npm version [<newversion> | major | minor | patch | premajor | preminor | prepatch | prerelease [--preid=<prerelease-id>] | from-git]`
 
 `npm version` is a command utility that it bumps the version of your applications and writes the new data back to `package.json`. 
 What's great about this cmd is that if run in a git repo, it will also create a version commit and tag by default. 
 
This can be used along custom npm scripts to create stable, safe, and standardized commits:

Example 
```
"scripts": {
  "preversion": "npm test",
  "version": "npm run build && git add -A dist",
  "postversion": "git push && git push --tags && rm -rf build/temp"
}
```
The exact order of execution is as follows:
```
1. Check to make sure the git working directory is clean before we get started. Your scripts may add files to the commit in future steps. This step is skipped if the --force flag is set.
2. Run the preversion script. These scripts have access to the old version in package.json. A typical use would be running your full test suite before deploying. Any files you want added to the commit should be explicitly added using git add.
3. Bump version in package.json as requested (patch, minor, major, etc).
4. Run the version script. These scripts have access to the new version in package.json (so they can incorporate it into file headers in generated files for example). Again, scripts should explicitly add generated files to the commit using git add.
5. Commit and tag.
6. Run the postversion script. Use it to clean up the file system or automatically push the commit and/or tag.
```

see resource links for full documentation

### git tags
Git supports two types of tags: lightweight and annotated.

A lightweight tag is very much like a branch that doesn’t change — it’s just a pointer to a specific commit.

Annotated tags, however, are stored as full objects in the Git database. They’re checksummed; contain the tagger name, email, and date; have a tagging message; and can be signed and verified with GNU Privacy Guard (GPG). It’s generally recommended that you create annotated tags so you can have all this information; but if you want a temporary tag or for some reason don’t want to keep the other information, lightweight tags are available too.

#### Creating tags
to create an annotated tag run:
`git tag -a v1.4 -m "my version 1.4"`

use `git show v1.4` to get the annotated details

to create a lightweight tag run:
`git tag v1.4-lw` (no flags are used here)

#### Listing tags
`git tag` with optional (`-l` or `--list`)

### Conclusion
(research not done)

Using `npm version` with `version` npm scripts, we should be able to keep and enforce a cohesive semantic versioning.
Not completely sure yet if this would be an equal replacement to npm versioning. We'd really need to make sure we handle
breaking commits/merges. To be discussed... 

### Resources
 - https://stackoverflow.com/questions/45338495/fetch-a-single-tag-from-remote-repository
 - https://docs.npmjs.com/cli/version
 - https://travishorn.com/semantic-versioning-with-git-tags-1ef2d4aeede6
 - https://git-scm.com/book/en/v2/Git-Basics-Tagging
 - https://travishorn.com/semantic-versioning-with-git-tags-1ef2d4aeede6
  
