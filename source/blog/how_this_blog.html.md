---
title: Deploying a static middleman site on Dokku
date: 1552137740
---

# Things to write, but no-where to write.. What to do?!

So here I am trying to get back into writing blogs. With no-where to write a blog. I've had the repository for this middleman site sitting around for about a half year now and had a day to kill. So I thought today seemed as good as any other day to have a play with Dokku.

## Why a static blog?

Mainly because it's just a fun project. But deploying a static site, means we aren't having to process a heck of a lot every time our webserver gets a request. Which makes it ideal to host on a low-end non-expensive instance.

## Dokku Dokku Panic!

Dokku is a docker powered mini-Heroku or Platform as a Service (PaaS) you can spin up in any virtual server you might have. It's pretty easy to set up yourself and even easier if you're willing to pay $5 a month to [DigitalOcean](https://www.digitalocean.com/) for one of their [one-click Dokku instances](https://www.digitalocean.com/docs/marketplace/dokku/).

If you have your own domain you can also do the cool heroku thing and host apps on subdomains off your domain. (blog.dekai.kiwi etc.)

Once you're up and running on Dokku the first thing we want to do is create our blog app.

```
dokku apps:create blog
```

and that's about it for now...

## There's just one problem...

Dokku (and also Heroku) don't really have an out of the box solution for deploying static HTML pages. So what can we do about this?

Well.. To be honest I just googled it and found this lovely gem to generate the files for me :)

[ngmaloney/middleman-dokku](https://github.com/ngmaloney/middleman-dokku)

But let's go through each of the files this gem generates and figure out what is going on...

**Procfile**

~~~~~~~~
web: bundle exec puma -p $PORT
~~~~~~~~

Pretty standard Heroku web worker definition here. A puma instance will be initialized for each `web` worker we choose to run. Each puma instance will run the below mentioned `config.ru`

**config.ru**

~~~~~~~~
# Modified version of TryStatic, from rack-contrib
# https://github.com/rack/rack-contrib/blob/master/lib/rack/contrib/try_static.rb

# Serve static files under a `build` directory:
# - `/` will try to serve your `build/index.html` file
# - `/foo` will try to serve `build/foo` or `build/foo.html` in that order
# - missing files will try to serve build/404.html or a tiny default 404 page

module Rack
  class TryStatic
    def initialize(app, options)
      @app = app
      @try = ['', *options.delete(:try)]
      @static = ::Rack::Static.new(lambda { [404, {}, []] }, options)
    end

    def call(env)
      orig_path = env['PATH_INFO']
      found = nil

      @try.each do |path|
        resp = @static.call(env.merge!({'PATH_INFO' => orig_path + path}))
        break if 404 != resp[0] && found = resp
      end

      found or @app.call(env.merge!('PATH_INFO' => orig_path))
    end
  end
end

use Rack::Deflater
use Rack::TryStatic, :root => "build", :urls => %w[/], :try => ['.html', 'index.html', '/index.html']

# Run your own Rack app here or use this one to serve 404 messages:
run lambda{ |env|
  not_found_page = File.expand_path("../build/404.html", __FILE__)
  if File.exist?(not_found_page)
    [ 404, { 'Content-Type'  => 'text/html'}, [File.read(not_found_page)] ]
  else
    [ 404, { 'Content-Type'  => 'text/html' }, ['404 - page not found'] ]
  end
}
~~~~~~~~
{: .language-ruby}

So looking at this, we can't strictly say that our site is 100% static, but it's good enough! This is just a simple web app to try and find an HTML file in the immediate file system. Specifically within the `build` directory

**Rakefile**

~~~~~
require 'middleman/dokku'

namespace :assets do
  task :precompile do
    sh 'middleman build'
  end
end

~~~~~

As part of the deployment process for a Ruby project. Dokku will attempt a `precompile` step. This is where we can define exactly what should happen at compile-time for our blog. In this case we want middleman to create the static HTML files in the `build` directory from the source directory

With that we should be ready to test the deploy!

## Beam me up Dokku

As with any deploy to Heroku. You will need to set up a new git remote for Dokku.

```
git remote add dokku dokku@<server-address>:<app-name>
```

Now all we need to do is give it a push and Dokku should take care of the rest!

```
git push dokku master
```

If everything goes well here. Then you should be seeing your Middleman site live in your Dokku app.

Congratulations! Now it's time for the victory lap!!

## Automated deploys

If you're anything like me you'll want to take as much hassle out of deployment of your blog as possible. So let's make it so that whenever we merge to master we automate a deploy to our Dokku instance.

This shouldn't be a problem for most CI solutions. I opted for CircleCI as it's the solution I'm most familiar with.

**.cirleci/config.yml**

~~~
version: 2
jobs:
  build:
    docker:
      - image: circleci/ruby:2.6.1
    working_directory: ~/repo
    steps:
      - checkout
      - run: bundle install --path vendor/bundle  # install dependencies

  deploy:
    machine:
        enabled: true
    working_directory: ~/repo
    steps:
      - checkout
      - run:
          name: Deploy Master to Dokku
          command: |
            git push $DOKKU_GIT_ENDPOINT master

workflows:
  version: 2
  build-and-deploy:
    jobs:
      - build:
          filters:
            branches:
              only: master

      - deploy:
          requires:
            - build
          filters:
            branches:
              only: master
~~~

`DOKKU_GIT_ENDPOINT` should be the entire git URI that you push to for deploys

Additionally we're going to need to generate some ssh credentials for our CI.

~~~
ssh-keygen

Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa): ~/.ssh/circleci

...
~~~

We need set the public key up on the Dokku instance.

```
cat ~/.ssh/circleci.pub | ssh root@<server-address> "sudo sshcommand acl-add dokku circle_deploy_key"
```

Then you'll need to set the private key in the CircleCI control panel for your project.

Now in theory whenever you push to master. You should build and deploy the latest version of your blog! Fancy!!

That's all for now. You can find the public repo for this blog and the specific file for this blog post below.

- [dekaikiwi/blog_middleman_template](https://github.com/dekaikiwi/blog_middleman_template)
- [how_this_blog.html.md](https://github.com/dekaikiwi/blog_middleman_template/blob/master/source/blog/how_this_blog.html.md)

~ Jono
