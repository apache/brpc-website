# brpc Website
The Web Site of brpc is based on [Apache Website Template](https://github.com/apache/apache-website-template).

# NOTE for PR
As this website is using the [gitpubsub](https://www.apache.org/dev/project-site.html) and we use Jekyll to generate the site.
We choose master branch to hold all the site source change and asf-site for apache gitpubsub.
Please sent your PR to the master branch instead of asf-site.

# How to run the site locally   
*  Change to non-root user  

*  Install [Ruby](https://www.ruby-lang.org/en/downloads/) and [Gem](https://rubygems.org/)   

*  Install Jekyll and Bundler   

   `sudo gem install jekyll bundler`  

*  Clone the site files

   `git clone https://github.com/apache/incubator-brpc-website.git`

* cd incubator-brpc-website

*  Install the gems with bundle

   `sudo bundle install`

*  Start the jekyll server

   `sudo bundle exec jekyll server`

*  Start web browser to access `http://localhost:4000`   

**Note that tested versions of the tools covered in this section are as following,**    

*  Ruby 2.6 
*  Gem 3.0   
*  Bundler 2.0   