#### Developing with `npm-link`
If your webapp relies on a dependency library (such as `@mlg/ui`) for production, but you want to develop them side by side
without pushing to git/npm after each change, you want to `npm link` them. Once linked, the linked project will reflect its updates
immediately in the source's node_modules.

* Note: for the purposes of this note, I will refer to our example `dep`endcy `lib`rary as `@mlg/ui` and our source project as `webapp`.

`app` = `webapp`

`lib` = `dep` = `@mlg/ui`


1. Go into `@mlg-ui` and run `npm-link`. A log should appear of the path where your module was linked to. `npm link` installs the package as a symbolic link in the system's global package location.
 Also add a watch script so that babel will compile on every change. Something like: `npx babel script.js --watch --out-file script-compiled.js`
 
 > `npm-link` should override any dependency already published

2. Go into `webapp` and run `npm-link @mlg/ui`. This won't add `@mlg/ui` to your dependency list but will add it 
to your `webapp/node_modules` where now any changes made to `@mlg/ui` will be automatically reflected in 
`webapp/node_modules`.

3. When you are done with your iteration of development you need to unlink the `dep` package from the webapp. You can do this 
by running `npm unlink @mlg/ui` inside the webapp. 
> **NOTE**: `npm unlink` is an alias for `npm uninstall` whereas `npm link`
is **NOT THE SAME** as `npm install`. You can alternatively remove `@mlg/ui` from your `webapp/node_modules` manually.

> **NOTE**: Your `dep` package will still be symlinked to your global folder. To remove the link globally run `npm unlink --global @mlg/ui`

> **NOTE**: `npm unlink` does not reinstall the original package that you overrid with the symlink. Make sure to `npm i @mlg/ui` again after you unlinked. Also, 
>`npm unlink` does not seem to remove the module from your node_modules, only your dependencies. Check to make sure.  
 
#### Immediate Issues
If `webapp` crashes after linking the `dep` module, first make sure your app is wrapped in an ErrorBoundary so we can get
more specific information: https://reactjs.org/docs/error-boundaries.html. Run a `console.error` in `componentDidCatch`.

1. Duplicate versions of React. See here: https://reactjs.org/warnings/invalid-hook-call-warning.html under `Duplicate React`. Also
se https://github.com/facebook/react/issues/13991

- in short, you want to link your `app`'s version of `react` to your `lib` so your `lib` is using the same `react` as your `app`.
Something like `npm link /absolute/path/to/webapp/node_modules/react` inside your `lib` module. 

But you most likely only need to do this. What happens is that when you link your lib to your app, the app's webpack is going to 
use the `node_modules` from that library's directory. Note that when you publish a module you don't publish the `node_modules` 
so there's no chance of webpack resolving from a different instance of a library. By adding the bottom lines to your webpack config,
you can (do either or both), alias a package name if you decided to put your `lib` inside of your `app` or if you symlinked, (in both
cases you have to do this), you have to have webpack resolve your `app`s node_modules as an override preference to any other node_modules 

- try adding 
```  
  resolve: {
    alias: {
        // only needed if your library is a git-module inside your app  
        '@mlg/ui': path.resolve('./src/libmods/mlg-ui/web')
    },
    // https://webpack.js.org/configuration/resolve/#resolvemodules
    modules: [path.resolve('node_modules'), 'node_modules']
  }
``` 
to your webpack config
                  
- or adding resolutions in your `package.json`. It's important here to note that this forces react to be the same version across
 dependencies. Beware if any of your deps need a specific dependency.
 
```  
"resolutions": {
   "**/react": "16.7.0-alpha.2",
   "**/react-dom": "16.7.0-alpha.2"
 },
```

#### Common Errors and Fixes
> ##### Errors: Node Version
- Make sure the linked project is under the same version of node that you are using for your app. Also make sure the same node version is being used to watch and run your apps

> ##### Errors: `exports is not defined`
- if you run into this error, add `modules: 'commonjs` to your `babel` config. 

> ##### ALIASING
- alias out `react`, `react-dom`, and `react-redux` if `modules: [path.resolve('node_modules'), 'node_modules']` doesn't work

> #### UNINSTALLING OR INSTALLING ANY MODULES WILL DESTROY YOUR LINK
- after you uninstall OR install any modules, it will destroy your link (removing most of its content) Make to to `re-link` after an `npm uninstall` of any kind
  
#### Resource links
* https://stackoverflow.com/questions/44515865/package-that-is-linked-with-npm-link-doesnt-update
* https://stackoverflow.com/questions/19094630/how-do-i-uninstall-a-package-installed-using-npm-link
* https://medium.com/dailyjs/how-to-use-npm-link-7375b6219557
* https://github.com/npm/npm/issues/6248
* https://docs.npmjs.com/cli/link.html
* https://github.com/facebook/create-react-app/issues/6027
* https://webpack.js.org/configuration/resolve/#resolvemodules

#### `npm help link`
```bash
> npm link
         alias: npm ln

   Description
       Package linking is a two-step process.

       First,  npm  link  in  a  package  folder  will  create  a symlink in the global folder {prefix}/lib/node_mod-
       ules/<package> that links to the package where the npm link command was executed. (see  npm-config  npm-config
       for the value of prefix). It will also link any bins in the package to {prefix}/bin/{name}.

       Next,  in some other location, npm link package-name will create a symbolic link from globally-installed pack-
       age-name to node_modules/ of the current folder.

       Note that package-name is taken from package.json, not from directory name.

       The package name can be optionally prefixed with a scope. See npm help scope.  The scope must be  preceded  by
       an @-symbol and followed by a slash.

       When  creating  tarballs  for  npm  publish,  the  linked packages are "snapshotted" to their current state by
       resolving the symbolic links.

       This is handy for installing your own stuff, so that you can work on it and test it iteratively without having
       to continually rebuild.

       For example:

             cd ~/projects/node-redis    # go into the package directory
             npm link                    # creates global link
             cd ~/projects/node-bloggy   # go into some other package directory.
             npm link redis              # link-install the package

       Now,   any   changes   to   ~/projects/node-redis   will   be  reflected  in  ~/projects/node-bloggy/node_mod-
       ules/node-redis/. Note that the link should be to the package name, not the directory name for that package.

       You may also shortcut the two steps in one.  For example, to do the above use-case in a shorter way:

         cd ~/projects/node-bloggy  # go into the dir of your main project
         npm link ../node-redis     # link the dir of your dependency

       The second line is the equivalent of doing:

         (cd ../node-redis; npm link)
         npm link redis

       That is, it first creates a global link, and then links the global installation  target  into  your  project's
       node_modules folder.

       Note  that  in  this  case,  you are referring to the directory name, node-redis, rather than the package name
       redis.

       If your linked package is scoped (see npm help scope) your link command must include that scope, e.g.

         npm link @myorg/privatepackage


```
