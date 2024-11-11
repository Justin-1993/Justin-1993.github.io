source "https://rubygems.org"

# Common gems for both environments
gem "jekyll-theme-chirpy", "~> 7.1", ">= 7.1.1"

# Conditionally include gems for local development
if ENV['LOCAL_DEV']
  gem 'jekyll-compose', group: [:jekyll_plugins]
  platforms :mingw, :x64_mingw, :mswin, :jruby do
    gem "tzinfo", ">= 1", "< 3"
    gem "tzinfo-data"
  end
else
  # Gems for GitHub Pages
  gem "html-proofer", "~> 5.0", group: :test
  platforms :mingw, :x64_mingw, :mswin, :jruby do
    gem "tzinfo", ">= 1", "< 3"
    gem "tzinfo-data"
  end
  gem "wdm", "~> 0.1.1", platforms: [:mingw, :x64_mingw, :mswin]
end
