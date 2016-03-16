# datafyit.github.io
Datafy.it Engineering Website

## Set it up locally
Clone this project
```bash
git clone https://github.com/datafyit/datafyit.github.io
```

If you don't have it yet, it's highly recommended to install [rvm](https://rvm.io/) first.

Use Ruby 2.1 or higher, and install bundler
```bash
gem install bundler
```
Afterwards you can install the gems required by running inside project root
```bash
bundle install
```

If all went well, you should be able to test if the site builds successfully (and will do so on github as well) by running
```bash
bundle exec jekyll build
```

To run the server (with live reload) use
```bash
bundle exec jekyll serve -w
```

To publish on the site, simply push on `master` and github will rebuild the site soon after.
