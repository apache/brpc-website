# brpc Website
The Web Site of brpc is based on [Apache Website Template](https://github.com/apache/apache-website-template).

# NOTE for PR
As this website is using the [gitpubsub](https://www.apache.org/dev/project-site.html) and we use Jekyll to generate the site.
We choose master branch to hold all the site source change and asf-site for apache gitpubsub.
Please sent your PR to the master branch instead of asf-site.

# How to run the site locally   

*  Install [Ruby](https://www.ruby-lang.org/en/downloads/) and [Gem](https://rubygems.org/)   

*  Install Jekyll and Bundler   

   `sudo gem install jekyll bundler github-pages`  

*  Clone the site files

   `git clone https://github.com/apache/incubator-brpc-website.git`

* cd incubator-brpc-website

*  Install the gems with bundle

   `sudo bundle install`

*  Start the jekyll server

   `sudo bundle exec jekyll server`

*  Start web browser to access `http://localhost:4000`   


# How to contribute
* Checkout the master branch to get the source, run locally 
* Modify and  create pull request as usual
* Switch to asf-site branch, copy the generated html from master branch(it is under target directory), then create pull request for asf-site branch.
* Wait for brpc.apache.org auto deploy works


**Note that tested versions of the tools covered in this section are as following,**    
(I tested it on Ubuntu)
*  Ruby 2.5 
*  Gem 2.7  
*  Bundler 2.0   