        const {src, dest, watch, series} = require('gulp');
        const browserSync = require('browser-sync').create();
        const sass = require('gulp-sass');
        const autoprefixer = require('gulp-autoprefixer');
        const cleanCSS = require('gulp-clean-css');
        const minify = require('gulp-minify');
        const htmlmin = require('gulp-htmlmin');
        const tinypng = require('gulp-tinypng-compress');

        function bs() {
          serveSass();
          browserSync.init({
            server: {
              baseDir: "./"
            }
          });
          watch("./*.html").on('change', browserSync.reload);
          watch("./sass/**/*.sass", serveSass);
          watch("./sass/**/*.scss", serveSass);
          watch("./js/*.js").on('change', browserSync.reload);
        };

        function serveSass() {
          return src("./sass/**/*.sass", "./sass/**/*.scss")
          .pipe(sass())
          .pipe(autoprefixer({
            cascade: false
          }))
          .pipe(dest("./css"))
          .pipe(browserSync.stream());
        };

        function buildCSS(done) {
          src('css/**/**.css')
          .pipe(cleanCSS({compatibility: 'ie8'}))
          .pipe(dest('project/css'));
          done();
        }

        function buildJS(done) {
          src(['js/**.js', '!js/**.min.js'])
          .pipe(minify({ext:{
            min:'.js'
            },
            noSource: true
          }))
          .pipe(dest('project/js'));
          src('js/**.min.js')
          .pipe(dest('project/js'));
          done();
        };

        function html(done) {
          src('**.html')
          .pipe(htmlmin({ collapseWhitespace: true }))
          .pipe(dest('project'));
          done();
        }

        function php(done) {
          src('**.php')
          .pipe(dest('project'));
          src('phpmailer/**/**')
          .pipe(dest('project/phpmailer'));
          done();
        }

        function fonts(done) {
          src('fonts/**/**')
          .pipe(dest('project/fonts'));
          done();
        }

        function imagemin(done) {
          src('img/**/**')
          .pipe(tinypng({key: 'n9QVLJ9kcmjpJf8fc6PZr5BB3jL1pYSk'}))
          .pipe(dest('project/img'));
          src('img/**/*.svg')
          .pipe(dest('project/img'));
          done();
        }

        exports.load = bs;
        exports.build = series(buildCSS, buildJS, html, php, fonts, /*imagemin*/);
