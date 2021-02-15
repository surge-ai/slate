
# Surge Documentation

### Getting Set Up

1. Install Ruby 2.5.7 `rvm install ruby-2.5.7`
2. Install gems `gem install bundler`
3. View the docs on a local server with `bundle exec middleman server` You should now be able to see the docs at http://localhost:4567.
4. Compile the docs to static HTML/CSS/JS in the build folder with `bundle exec middleman build --clean`

Detailed instructions are [here on the Slate Wiki](https://github.com/slatedocs/slate/wiki/Using-Slate-Natively)


### Updating docs

1. Run `bundle exec middleman build --clean` to compile docs into the `build` directory. 
2. Copy the newly generated fonts, image, javascripts, and css folders from `build` to `gondor/public/docs/`
3. Copy the contents of build/index.htmll to `gondor/app/views/website/human_api_docs.html.erb`
4. Replace all instances of "{{YOUR_API_KEY}}" with <%= @api_key %>. so people can see their own API key on the docs if they are logged in.
5. Edit the TOC foot section to check if the current user is logged in. The updated version looks like this:

```
  <% unless current_user %>
    <ul class="toc-footer">
      <li><a href='https://www.app.surgehq.ai/poster_sign_up'>Sign up for an account</a></li>
    </ul>
  <% end %>
  ```