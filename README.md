# Setup

Install `rbenv`:

```shell script
# install rbenv
brew install rbenv

# initialize rbenv
rbenv init

# reload the terminal
source ~/.zshrc

# install a fixed version of ruby
rbenv install 2.7.2
rbenv global 2.7.2

# restart the terminal

# install Bundler and Jekyll
gem install --user-install bundler jekyll

# Add gem binary path to zshrc: 
echo 'export PATH="$HOME/.gem/ruby/2.7.0/bin:$PATH"' >> ~/.zshrc

# reload the terminal
source ~/.zshrc

# Install dependencies
bundle install

# Run the local server
bundle exec jekyll serve
```