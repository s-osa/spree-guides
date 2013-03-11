Deploying your Application to Heroku
------------------------------------

This article will walk you through configuring and deploying your Spree
application to Heroku.

This guide assumes that your application is deploy-ready and that you
have a Heroku application already created on the Heroku stack for this
application. If you don’t have a Heroku app already, follow [this
guide](https://devcenter.heroku.com/articles/creating-apps).

endprologue.

### Configuring your application

There are two major things you will need to edit before you can deploy
your application to Heroku: the asset pipeline’s
*initialize\_on\_precompile* setting and S3 settings.

#### Asset Pipeline

When deploying to Heroku by default Rails will attempt to intialize
itself before the assets are precompiled. This step will fail because
the application will attempt to establish a database connection, which
Heroku will not have set up yet.

To work around this issue, put this line underneath the other
*config.assets* lines inside *config/application.rb*:

<ruby>\
config.assets.initialize\_on\_precompile = false\
</ruby>

The assets for your application will still be precompiled, it’s just
that Rails won’t be intialized during this process.

#### S3 Support

Because Heroku’s filesystem is readonly, you will need to configure
Spree to upload the assets to an off-site server, such as S3. If you
don’t have an S3 account already, you can [set one up
here](http://aws.amazon.com/s3/).

This guide will assume that you have an S3 account already, along with a
bucket under that account for your files to go into, and that you have
generated the access key and secret for your S3 account.

To configure Spree to upload images to S3, put these lines into
*config/initializers/spree.rb*:

<ruby>\
Spree.config do |config|\
 config.use\_s3 = true\
 config.s3\_bucket = ‘<bucket>’\
 config.s3\_access\_key = “<key>”\
 config.s3\_secret = “<secret>”\
end\
</ruby>

If you’re using the Western Europe S3 server, you will need to set two
additional options inside this block:

<ruby>\
Spree.config do |config|\
 …\
 config.attachment\_url = “:s3\_eu\_url”\
 config.s3\_host\_alias = “s3-eu-west-1.amazonaws.com”\
end\
</ruby>

And additionally you will need to tell paperclip how to construct the
URLs for your images by placing this code outside the *config* block
inside *config/initializers/spree.rb*:

<ruby>\
Paperclip.interpolates(:s3\_eu\_url) do |attachment, style|\

“\#{attachment.s3\_protocol}://\#{Spree::Config[:s3\_host\_alias]}/\#{attachment.bucket\_name}/\#{attachment.path(style).gsub(%r{\^/},”“)}”\
end\
</ruby>

### Pushing to Heroku

Once you have configured the above settings, you can push your Spree
application to Heroku:

<shell>\
git push heroku master\
</shell>

### SSL Support

For information about SSL support with Heroku, please read their [SSL
Guide](https://devcenter.heroku.com/articles/ssl).
