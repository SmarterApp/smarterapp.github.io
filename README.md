Source to the www.SmarterApp.org website. Uses GitHub Pages and Jekyll.

Running Jekyll Locally
======================
To run the Jekyll server locally, do the following:

Ruby Setup/Configuration
------------------------
* Determine ruby version:
  * `ruby --version`
  * Example output: `ruby 2.1.6p336 (2015-04-13 revision 50298) [x86_64-darwin12.0]`
* If ruby is not installed OR is not at least v 2.0.0, instructions for installing or updating ruby can be found [here](https://www.ruby-lang.org/en/downloads/)
* Install the [`bundler`](http://bundler.io/) gem (if it isn't already installed):
  * `gem install bundler`

Run The Web Server Locally
--------------------------
* Clone this repository:
  * `git clone [repository url]`
* Navigate to the directory where the repository exists:
  * `cd /path/to/local/repository`
* Run the following command to install the dependencies:
  * `bundle install`
* Start the jekyll server (which also builds the site for the first time):
  * `jekyll serve`
* The startup output should appear as follows:
```
Configuration file: /Users/jeffjohnson/dev/sbac/git/smarterapp.github.io/_config.yml
Configuration file: /Users/jeffjohnson/dev/sbac/git/smarterapp.github.io/_config.yml
            Source: /Users/jeffjohnson/dev/sbac/git/smarterapp.github.io
       Destination: /Users/jeffjohnson/dev/sbac/git/smarterapp.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 4.763 seconds.
 Auto-regeneration: enabled for '/Users/jeffjohnson/dev/sbac/git/smarterapp.github.io'
Configuration file: /Users/jeffjohnson/dev/sbac/git/smarterapp.github.io/_config.yml
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```
* Navigate to `http://localhost:4000` in a browser

Additional References
---------------------
* [GitHub Pages Setup Documentation](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/)
* [Jekyll Installation Guide](https://jekyllrb.com/docs/installation/)
* [Jekyll Troubleshooting Guide](http://jekyllrb.com/docs/troubleshooting/#jekyll-amp-mac-os-x-1011)