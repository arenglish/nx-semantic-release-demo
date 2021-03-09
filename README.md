# Example Angular Application Monorepo Semantic Release Workflow

This is an example Nx monorepo workspace of angular applications that only publishes affected applications using nx's `affected` commands, a custom project target, and semantic release.

I have not tested this yet in a public CICD job, but have run it successfully locally with `semantic-release --ci=false`

Please post an issue for any questions or if it does not work for you.

This does not yet demonstrate how to:
* update an inner project Changelog with only commits relevant to that project (semantic-release-monorepo pacakge has that capability, but I have not tried it in an nx workspace yet)
* release applications on independent versions (this method is platform versioned, yet only publishes applications that have changed per version)

* Add a "release" target to your apps "architect" property in angular.json
```json
{
"release": {
  "builder": "@nrwl/workspace:run-commands",
  "options": {
    "commands": [],
    "cwd": "./",
    "parallel": false
  }
}
}
```
* Configure release commands to version an inner package.json asset, build the app, and npm publish
```json
{
"build": {
    "assets": [
      "apps/notes/src/favicon.ico",
      "apps/notes/src/assets",
      {
        "glob": "package.json",
        "input": "apps/notes",
        "output": "/"
      }
    ]
},
"release": {
  "builder": "@nrwl/workspace:run-commands",
  "options": {
    "commands": [
      "cd apps/calendar && npm version {args.version}",
      {
        "command": "ng build calendar --prod",
        "forwardAllArgs": false
      },
      {
        "command": "cd dist/apps/calendar && npm publish",
        "forwardAllArgs": false
      }
    ],
    "cwd": "./",
    "parallel": false
  }
}
}
```
* In semantic release config file, use exec plugin to run your new nx run-commands, passing in the version to be released
```json
["@semantic-release/exec", {
  "publishCmd": "nx run affected --target=release --args=\"--version=${nextRelease.version}\""
}]
```

TODO:
* add versioned files to @semantic-release/git assets so they are commited into repo
