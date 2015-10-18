title: Site deployment with gzipped assets to AWS S3 using CircleCI
date: 2015-04-16 18:02
tags: 
- aws
- automation
categories: 
- tech
---

I've been using [Heroku](https://www.heroku.com) and [Google App Engine](https://cloud.google.com/appengine/docs) to host most of my web applications. This time, I decided to use [Amazon S3](http://aws.amazon.com/s3) to serve a static website.

## Amazon S3 setup
After creating your S3 bucket, click on the `Properties` button.

1) Enable static website hosting. For single page apps, specify the same error and index document.

![s3_static_config](https://alyssaq.github.io/blog/images/S3deploy_enable-static-site.png)

2) Under Permissions > Edit CORS Configuration, make all files in this bucket accessible to the public by pasting this XML:

    <?xml version="1.0" encoding="UTF-8"?>
    <CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
        <CORSRule>
            <AllowedOrigin>*</AllowedOrigin>
            <AllowedMethod>GET</AllowedMethod>
            <MaxAgeSeconds>3000</MaxAgeSeconds>
            <AllowedHeader>Authorization</AllowedHeader>
            <AllowedHeader>Content-Type</AllowedHeader>
        </CORSRule>
    </CORSConfiguration>

There are 2 save buttons. Per section and global save, so click em all!

## Gzip assets with gulp
Unfortunately, S3 does not automatically compress assets. Here is their [verbose instructions for serving compressed files](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/ServingCompressedFiles.html#CompressedS3).  

Basically,    

1. gzip assets
2. Set header with `Content-Encoding: gzip`

Yes, my JS and CSS assets are always gzipped and ancient browsers (such as IE < 5, Firefox < 0.9.5) are not supported. Code for the future!

I gzip as part of my [gulp](http://gulpjs.com) build.

Heres the relevant gulp `build` task:

<pre><code class="language-javascript">
gulp.task('build', ['clean'], function () {
  gulp.start('default', function() {
    return gulp.src('dist/**/*')
      .pipe($.size({title: 'build', gzip: true}))
      .pipe($.if('*.js', $.gzip({ append: false })))
      .pipe($.if('*.css', $.gzip({ append: false })))
      .pipe(gulp.dest('dist'))
  })
})
</code></pre>

I run `clean` (to delete destination folder), then the `default` task (to browserify, babelify, concatenate, minify, replace, etc) and then `gzip` only the minified JS and CSS assets.

`$.gzip({ append: false })` replaces the file with the gzipped version and pipes to the `dest` folder. Setting to true appends `.gz`.

The npm gulp packages involved in this step:

    $ npm install gulp-load-plugins gulp-if gulp-gzip 

## Auto-deploy with CircleCI
In the CircleCI project settings > AWS Permissions, set your Amazon `Access Key ID` and `Secret Access Key` from an IAM user with S3 PutObject and GetObject permissions ([Amazon S3 permission list](http://docs.aws.amazon.com/AmazonS3/latest/dev/using-with-s3-actions.html)). I started off with a user that had permissions to all S3 actions and then pruned later.

To deploy the app to S3, circleCI runs `gulp build` and sync the `dist` folder with [AWS CLI sync](http://docs.aws.amazon.com/cli/latest/reference/s3/sync.html).

My `circle.yml`:

<pre class="language-yml"><code class="language-yml">
machine:
  node:
    version: 0.12.0
dependencies:
  post:
    - npm install -g gulp-cli && gulp build
deployment:
  production:
    branch: master
    commands:
      - sudo pip install awscli
      - aws s3 sync dist/ s3://newcleus-app --exclude "*" --include "*.css" --include "*.js"
 --content-encoding gzip --cache-control public,max-age=30672000
      - aws s3 sync dist/ s3://newcleus-app --exclude "*.js" --exclude "*.css"
</code></pre>
           
Under `deployment`, the first `aws s3` command adds the `Content-Encoding: gzip` and `Cache-Control` headers to the JS and CSS assets. The second command syncs the remaining files.

## Overall
On my Mac, I use [Cyberduck](https://cyberduck.io) to view all files in all my S3 buckets.

In the end, while not as friendly as Heroku, I was pretty happy with this auto-deployment to S3.