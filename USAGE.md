## Usage

- Install `minify-html-attributes` from `npm` and include it in your Node.js-based build script.
- Copy all your source HTML, CSS, and JS files to a location of your choice that is safe for editing.
- Run them through `minify-html-attributes`. `minify-html-attributes` will return an object that is a list of files that it modified. Files that were not modified will not be included in the object.
- Update the copied files with the new changes.
- Run any additional minifiers you like against the modified files as normal.

### Sample app

Here's a full example that implements `minify-html-attributes` prior to running webpack and starting an Express server:

```javascript
const fs = require('fs-extra')

// copy all source html, css, and js to locations that are safe for editing
fs.copySync('mvc/views', 'mvc/.preprocessed_views') // copy unmodified html to a modified templates directory
fs.copySync('styles.css', '.preprocessed_statics/styles.css') // copy css file to the public folder
fs.copySync('browser.js', '.preprocessed_statics/browser.js') // copy css file to the public folder

// load minify-html-attributes
const editedFiles = require('minify-html-attributes')({
  htmlDir: './mvc/.preprocessed_views', // where your html files to process are located
  cssDir: './.preprocessed_statics', // where your css files to process are located
  jsDir: './.preprocessed_statics' // where your js files to process are located
})

// update any file that was postprocessed
for (const fileName in editedFiles) {
  const editedFile = editedFiles[fileName]
  fs.writeFileSync(fileName, editedFile.contents)
}

// copy preprocessed assets to final location

// html: no need for html files because they are edited in-place in ./mvc/.preprocessed_views

// css: just copy the file to the public folder; you could also do something fancier like run it through a preprocessor
fs.copySync('.preprocessed_statics/styles.css', 'public/styles.css') // copy css file to the public folder

// js: bundle preprocessed js after it is preprocessed
const webpack = require('webpack')
const path = require('path')
const webpackConfig = {
  mode: 'production',
  entry: './.preprocessed_statics/browser.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'public'),
    libraryTarget: 'umd'
  },
  devtool: 'source-map'
}
const compiler = webpack(webpackConfig)
compiler.run((err, stats) => {
  if (err || stats.hasErrors()) {
    console.error('Webpack build failed:', err || stats.toJson().errors)
  } else {
    // configure express
    const express = require('express')
    const app = express()
    app.use(require('body-parser').urlencoded({ extended: true })) // populates req.body on requests
    app.engine('html', require('teddy').__express) // set teddy as view engine that will load html files
    app.set('views', 'mvc/.preprocessed_views') // set template dir
    app.set('view engine', 'html') // set teddy as default view engine
    app.use(express.static('public')) // make public folder serve static files

    // load express routes
    require('./mvc/routes')(app)

    // start express server
    const port = 3000
    app.listen(port, () => {
      console.log(`🎧 express sample app server is running on http://localhost:${port}`)
    })
  }
})
```

Here's how to run the sample app:

- `cd sampleApps/express`
- `npm ci`
- `cd ../../`
- `npm run express-sample`
  - Or `npm run sample`
  - Or `cd` into `sampleApps/express` and run `npm ci` and `npm start`
- Go to [http://localhost:3000](http://localhost:3000)
