# Part 18: [gulp](https://github.com/gulpjs/gulp), _The Streaming Build System_


At its simplest, gulp is a build system and collection of plugins using [NodeJS streams](https://github.com/substack/stream-handbook#introduction). The idea is to read in a collection files, pass them through a series of modules, and then finally write out the result.

## Why use gulp?

 - __Avalaible Plugins__: By having modules do one thing well, gulp allows for easily reused modules to be added, reordered and removed without effecting those up/downstream. Leaving a flexible and versatile collection of jobs.

- __Tasks Are Easily Chained Together__: The use of streams is highly agreeable when trying to build a web app for a production environment where a series of improvements (minification, concatenation, etc )  and checks (linting, unit tests, etc) need be preformed and then outputted to a build folder. Chaining together the different modules together to take in the source files, preform an improvement/transformation, and then output the resulting build to a build folder, will create a repeatable automated build.

- __Same Tech Stack as The Rest of the Project__: No need to use another other language, gulp tasks are coded in NodeJS.

- __Development Tasks are Stored Within the Project__: Packages providing gulp tasks  can/should be tracked in _package.json_ -> _"devDependencies"_. New developers or CI jobs just need to run ```npm install``` to have all the required build/test dependencies installed.

- __Cross Platform__: Runs on Linux, Windows, and OSX.

## Examples
The below preforms a subset of the same tasks as the included gulpfile.js but rangle-gulp tasks have been decapsulated for clarity.



## ```gulp karma```
Stream in  all client app scripts and external dependencies and run the client unit tests. The test results are outputted to the terminal (according the reporter defined in ```karma.conf.js```). This will break the stream if one or more tests fail.
```javascript
var gulp = require('gulp');
var karma = require('gulp-karma');
gulp.task('karma', function(){  
  return gulp.src(/*GLOB matching all client app, test, and required dependancy files */)
  .pipe(karma({
    configFile: 'client/testing/karma.conf.js',
    action: 'run'
  }))
  .on('error', function (err) {
    // Make sure failed tests cause gulp to exit non-zero
    throw err;
  });
});
```

## ```gulp lint```
 Lint the client app js files. [JSHint](http://jshint.com/about/) will scan all the source JS files for errors or code inconsistencies (as defined in ```.jshintrc```), and will log and break the stream if any linting errors were found.
```javascript
var gulp = require('gulp');
var jshint = require('gulp-jshint');
gulp.task('lint', function () {
  return gulp.src(/*GLOB matching all client app scripts*/)
  .pipe(jshint())
  .pipe(jshint.reporter('default'))
  .pipe(jshint.reporter('fail'));
});
```

## ```gulp beautify```
 Beautify the source client script files. [JSBeautify](http://jsbeautifier.org/) will reformat the JS files according the rules defined in ```.jsbeautifyrc```. The goal is to have consistent indentation, spacing, line length, and bracket placement  across the entire codebase.
```javascript
var gulp = require('gulp');
var beautify = require('gulp-js-beautify');
var fs = require('fs');

gulp.task('beautify',  function () {
  var jsBeautifyConfig = JSON.parse(fs.readFileSync('.jsbeautifyrc'));
  return gulp.src(/*GLOB matching all client app scripts*/, { base: '.' })
    .pipe(beautify(jsBeautifyCwonfig))
    //overwrite the inputted files so the changes will be committed to the project
    .pipe(gulp.dest('.'));
});
```

## ```gulp concatAndMinify```
 Combine and minify all the client script files into all.min.js then output the result to the build folder. By combing all files into a single file, the app will be loaded quicker due to fewer number of round trips to the server to load assets. By minifying the concatenated file, the file size is significantly reduced.
```javascript
var gulp = require('gulp');
var gulpFilter = require('gulp-filter');
var concat = require('gulp-concat');
var rename = require('gulp-rename');
var uglify = require('gulp-uglify');
gulp.task('concatAndMinify', function () {  
  return
    gulp.src(/*GLOB matching all client app scripts*/)
    .pipe(gulpFilter(/*functuion to filter out all test files*/))
    .pipe(concat('all.js'))
    .pipe(gulp.dest(/*build folder path*/))
    .pipe(rename('all.min.js'))
    .pipe(uglify())
    .pipe(gulp.dest(/*build folder path*/));
  };
);
```

## ```gulp assets```
Copy assets (html, css, images, etc) into the build folder.
```javascript
gulp.task('assets',function(){
  return gulp.src( [ /*GLOB(s) to include all html, css, fonts, images, etc*/ ],
   { 'base' : './client' })
    .pipe(gulp.dest(/*build folder path*/))
});
```


## ```gulp dev```
Start a server to serve the client app files and auto reload the page when any client file changes (no longer have to manually reload the page to see changes). Requires the related browser [app](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei?hl=en). The client app will be accessible at http://localhost:8080/
```javascript
var gulp = require('gulp');
var connect = require('gulp-connect');

//initalize the server to serve the client app
function initConnect(options) {
  connect.server(options);
  watch({ glob: options.glob })
  .pipe(connect.reload());
  return connect.reload();
};

gulp.task('dev', initConnect({
  root : './client', //which folder to serve
  port : 8080,
  livereload : true, //reload the page when a client file is changed
  // Files to watch for live re-load
  glob : [/* GLOB(s) to cover all client files*/]
}));
```

## ```gulp```
The task that is run when no task is specified. If all the above examples are included in our ```gulpfile.js```, this will run the series of already defined tasks. It will lint, then run unit test,  concat and minify the app scripts, then copy the asset files over. The end result will be a build folder will a ready to be deployed version of the current app.
```javascript
gulp.task('default', ['lint', 'karma', 'concatAndMinify', 'assets']);
```
