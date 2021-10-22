---
layout: post
title: bundle exec rails vs bin/rails
---

I have noticed that to launch rails console I would sometimes type:
`bin/rails c` and other times `bundle exec rails c`.

Both of the commands work and that's why I guess I have been using them interchangably. So why do we have these two options then, I wondered?

### Here is what I found
`bundle exec rails` runs the rails command through the bundler which ensures that the command is run in the application context, that is with all the gem versions specified in Gemfile.lock. It enforces the gem versions by setting the `$LOAD_PATH` variable with paths to all the gem versions specified in your `Gemfile.lock`. Ruby then uses `$LOAD_PATH` to  load all the gems that are to be used in the code.

With `bin/rails` you are not calling Bundler directly, instead you are executing a [binstub](https://bundler.io/man/bundle-binstubs.1.html#DESCRIPTION) from your project's bin directory. Similar to `bundle exec`, the binstub first loads the context of Gemfile.lock by running `bundler/setup`, but also calls `bootsnap/setup` (since Rails 5) that speeds up loading of the application [through caching](https://github.com/Shopify/bootsnap#how-does-this-work), and then loads the original rails executable from the rails gem.

### Which one to use?
Since bundler is independent of Rails, and is used with other Ruby projects, you will probably always have the two options. I prefer using the `bin/rails` and other binstubs now, because it's more explicit that I am trying to run the rails (or other executable) of the application in which those binstubs exist. `bin/rails` is also faster because of the bootsnap caching and skipping of bundler's bundling steps. Last but not least, `bin/rails` is shorter than `bundle exec rails`. 🙃

### What about just *rails c*
Never run just `rails` or `rake` commands, without specifying either `bundle exec` or the path to a binstub. This would just use whatever rails version is set to be the default version on your system which might differ from the version used in the project.
