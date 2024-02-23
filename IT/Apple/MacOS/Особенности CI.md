![[Pasted image 20240502021648.png]]


обновление до fasltane, 2.220 танет за собой вверх все другие зависимости (гемы)



The bundle exec command is used with Bundler to run commands that use a project's gem dependencies within the context of the Bundler-managed environment.



pod deintegrate && pod cache clean --all && pod install


https://rollbar.com/blog/ruby-bundle-install-errors/#

 By understanding the causes of the errors and using the methods mentioned below, we can easily troubleshoot and fix bundle install errors in our Ruby projects:

    Run gem update --system to update RubyGems to the latest version.
    Run gem install bundler to install or update the Bundler gem.
    Run bundle update to update all gems to their latest versions.
    Check the Gemfile.lock file and try to remove or update the conflicting gems.
    Run bundle install --verbose to get specific error messages that might give you further insight into the problem.


