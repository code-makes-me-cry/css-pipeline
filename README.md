# CSS pipeline

The included `package.json` file contains handy scripts that process CSS from dev to production as follows:

Sass => minified CSS => Autoprefixed CSS for whichever browsers you want to support

## How can this help me?

This pipeline hopes to make these workflow dreams come true:

- We want to write Sass because it has nicer features than plain CSS.
- We want to write modern CSS but still support older browsers.
- We've heard that [vendor prefixes](https://autoprefixer.github.io/) can help with browser support ([with limitations](https://css-tricks.com/css-grid-in-ie-css-grid-and-the-new-autoprefixer/#autoprefixer-still-cant-save-you-from-everything)), but we want the prefixing to be handled automatically, so we don't have to think about it.
- We want our code to run as fast as possible on the user's machine, so we want to make sure it's compressed it (i.e. [minified](https://www.imperva.com/learn/performance/minification/)).

If that doesn't quite sound like your level yet, and some of those ideas are unfamiliar, check out the links to starter resources at [codemakesmecry.com](https://www.codemakesmecry.com/posts/css-pipeline)

## Prerequisite: NPM

This pipeline uses **npm scripts**, which you might not have come across before, but this is a great time to learn about them.

NPM is a foundational tool that will be useful across many different flavors of web dev projects, so it's well worth getting acquainted with now. As with most things, you can get started with the basics, then skill up gradually as you use it on real projects over time.

If you've never heard of a `package.json` file, or if you've forgotten what it's all about, I'd recommend this [npm tutorial for beginners](https://www.youtube.com/watch?v=6fj0cpmMiVg&list=PLC3y8-rFHvwhgWwm5J3KqzX47n7dwWNrq&ab_channel=Codevolution) (it's an hour-ish long YouTube playlist).

## Overview

1. Copy `package.json` into your project's root directory. Update the package's **name** and any other details about your project, if you like.

2. Check that your file structure matches the paths in the scripts (read on for details).

3. Install the latest versions of the tools needed for this pipeline (read on for details).

## File system starting point

This pipeline uses [npm scripts](https://docs.npmjs.com/cli/v6/using-npm/scripts) that are provided in [`package.json`](./package.json). If you want them to work out of the box, please set up your file structure as follows. If you want to use another file structure, you may need to edit the paths in each script.

Here's what my project directory looks like. The parts you'll need to create manually are `src/` (plus everything in it) and `index.html`.
<!-- TODO: provide boilerplate -->

`main.scss` will have one job, which is to load [Sass partials](https://sass-lang.com/guide#:~:text=A%20partial%20is%20a%20Sass,used%20with%20the%20%40use%20rule.). No style rules are in this file.
Style rules are defined in partials.
This example below has just one partial, `_base.scss`, but a real-world project will usually have more.

The `modules/` dir is optional. We can organise our Sass partials however we like, depending on what structure makes life easier for each project. People have different ways they like to do this, and many have published articles about it. For ideas, Google *sass file structure something something*. With practice, you'll find what works for you.

```fs
.
├── README.md
├── index.html
├── node_modules/
├── package-lock.json
├── package.json
└── src
    └── scss
        ├── main.scss
        └── modules
            └── _base.scss
```

## Usage

### Install (do this once at the start of each project)

#### Understand what we're about to install

We are going to install the following packages as "dev dependencies", because they're tools that we only need during development (our users never have to run them, otherwise they would just be regular "dependencies").

I'm going to list them in the order that makes it easier to explain, but you may find them listed alphabetically elsewhere.

| Package        | Why         | Docs worth reading |
| :------------- | :---------- | :----------------- |
| `dart-sass`    | Takes our Sass and compile it into CSS | [Sass](https://sass-lang.com/) |
| `autoprefixer` | Creates new CSS with the required prefixes, based on our CSS and our list of supported browsers | [Autoprefixer](https://github.com/postcss/autoprefixer/blob/master/README.md) |
| `postcss`      | Allows autoprefixer to do its job |
| `postcss-cli`  | Allows us to use an npm script to handle autoprefixing |

#### Install stuff

```console
$ npm install --save-dev dart-sass autoprefixer postcss-cli postcss
```

#### Check it worked

This should update `package.json` to include devDependencies similar to the following (your version numbers may vary, since the above command will install the latest):

```json
  "devDependencies": {
    "autoprefixer": "^10.0.3",
    "dart-sass": "^1.25.0",
    "postcss": "^8.1.10",
    "postcss-cli": "^8.3.0",
  }
```

### Dev script (keep this running during development)

```console
$ npm run dev
```

Every time `main.scss` changes (or any of the partials it's loading), compile SCSS to CSS with a source map.

Run this in a terminal that you keep open, so it can run in the background and actively watches for every time you hit 'save' on your Sass files. When you need to stop the process, hit `Ctrl + c` (or close the terminal window). You should stop the process when you're happy with how things look, and you don't need to make any more style changes.

This script creates two files at the project root (the same folder that contains `index.html` above), based on the contents of `src/scss/main.scss`:

- style.css
- style.map.css

The source map `style.map.css` is useful so that our browser dev tools can tell us exactly which Sass files are responsible for each style. This way, when we need to make a change, it's easy to see where to do that.

Without the source map, the dev tools would just give us a very unhelpful "lol I just got the style from style.css. You figure out which Sass file, buddy."

In `index.html`, it is enough to just link to `style.css`. The browser will detect the source map automatically.

### Build scripts (run once before shipping to production)

```console
$ npm run build
```

This one command will run two scripts in sequence:

1. prebuild script
  a. Remove files that mess with Autoprefixer: `style.map.css`
  b. Compile Sass to a new temporary file: `src/no-prefix.css`
  c. Make sure that the output is in a [minified format](https://teamtreehouse.com/community/could-someone-please-give-an-example-of-minified-css).

2. build script
  Use Autoprefixer to take all the styles from `src/no-prefix.css` (created in the previous step) and create a new file with added prefixes at `style.css` (this overwrites the one generated by `npm run dev`).

  Prefixes are added depending on CSS features supported by the browsers that are listed in `package.json` under `browserslist`. We should update this list according to the browser support required for each project. See the [Browserslist docs](https://github.com/browserslist/browserslist/blob/master/README.md) to learn what kinds of values are valid here.

  <!-- TODO: update build script to minify the output -->

## Troubleshooting

### **Issue:** Styles won't update while Sass is in watch mode

Often this happens after Sass encountered an error (e.g. the file was saved while you were in the middle of writing something, or there was a syntax error).

After you fix the error, you may see that the error is resolved in the console, but silently Sass will stop updating your styles in the browser.

To fix it, **get into the habit of manually stopping and restarting the watch process every time Sass gives you an error**, even if the console looks like it's working again, because often it's not.

### **Error message:** rm: ./style.css.map: No such file or directory
If you get the above console error after running ```npm run build```

It might be because ```npm run dev``` creates a file that ```npm run build``` expects to delete.
<!-- TODO: update build script to first check if files exist before attempting to remove them -->

To fix it, try this:

1. Run ```npm run dev```
2. Save one of your Sass files so that `style.map.css` is automatically created
3. Stop ```npm run dev```
4. Try again to run ```npm run build```

If you run into problems and Google is no help, please feel free to [file an issue](https://docs.github.com/en/free-pro-team@latest/github/managing-your-work-on-github/creating-an-issue).
