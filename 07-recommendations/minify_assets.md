# Minify Assets

If you aren't already, start using minifiers for your static assets. Many very good ones are available (as discussed in the chapter on minification). To set up a very basic minification system, you can follow the following instructions. As your project matures, it's often common to use a build system like Grunt, Gulp, or simple Makefiles.

These instructions assume that `./node_modules/bin` is on your `PATH` environment variable.

```bash
# Install the tools needed for a basic build system
npm install uglify-js clean-css

# Concatenate your JavaScript into a single file
# Change `static/` to the path to your assets
find static/ -name "*.js" -exec cat {} \; > static/main.js

# Minify your concatenated JavaScript
# -m performs name mangling, -c performs certain optimizations
uglifyjs -m -c static/main.js > static/main.min.js

# Concatenate your CSS into a single file
find static/ -name "*.css" -exec cat {} \; > static/styles.css

# Minify your concatenated CSS
cleancss static/styles.css > static/styles.min.css
```

After minifying, be sure to update your application to use the minified files instead of the unminified versions.