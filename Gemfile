# frozen_string_literal: true

source "https://rubygems.org"
gemspec

gem 'jekyll-archives-hofill'
#!/usr/bin/env bash
set -e # halt script on error

bundle exec jekyll build
touch ./_site/.nojekyll # this file tells gh-pages that there is no need to build
