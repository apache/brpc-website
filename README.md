# brpc Website 
The Website of brpc (http://brpc.apache.org) is using the apache.org template from  [Apache Website Template](https://github.com/apache/apache-website-template).

# Note for PR
As this website is using the [gitpubsub](https://www.apache.org/dev/project-site.html) and we use Jekyll to generate the site.
We choose master branch to hold all the site source change and asf-site for apache github website.
Please sent your PR to the master branch instead of asf-site.

# How to run the site locally   

*  Install [Ruby](https://www.ruby-lang.org/en/downloads/) and [Gem](https://rubygems.org/)   

   `brew install ruby`

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
* Checkout the master branch to get the source, run locally to generate html content
* Modify and  create pull request as usual
* Switch to asf-site branch, copy the generated html content from master branch( They are under target directory), then create pull request for asf-site branch.
* Wait for brpc.apache.org auto deploy works


**Note that tested versions of the tools covered in this section are as following,**    
(I tested it on MacOS)
*  Ruby 2.6.3p62 
*  Gem 3.0.3  
*  Bundler 2.1.4 
