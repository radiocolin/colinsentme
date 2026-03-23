source "https://rubygems.org"

# Avoid gems with problematic native extensions (eventmachine, bigdecimal, etc.)
gem "jekyll", "~> 4.3"

# We pin the sass converter to 2.x to avoid sass-embedded/protobuf/bigdecimal
gem "jekyll-sass-converter", "~> 2.0"

group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.17"
  gem "jekyll-seo-tag", "~> 2.8"
  gem "jekyll-sitemap", "~> 1.4"
end

# Webrick is required for Ruby 3.0+
gem "webrick"

# Explicitly use a version of eventmachine that builds correctly if needed,
# or avoid it by ensuring Jekyll doesn't pull in em-websocket.
# Actually, Jekyll 4 still pulls in em-websocket for 'serve'.
# Let's try to force a newer eventmachine that has the fix.
gem "eventmachine", ">= 1.2.7"
