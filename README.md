# ivan_the_terribles_blog
By [Scott Macpherson](https://github.com/scottmacphersonmusic)

### Description
Ivan's blog was once terribly insecure, but with recent improvements it is now significantly more secure.
### Improvements
##### SQL Injection
Initially Ivan's blog was vulerable to SQL injection attacks like pasting the following in the address bar:
```
http://localhost:3000/posts?utf8=%E2%9C%93&search=archive&status=foo=%22bar%22%3E%3Cscript%3Ealert%28%22p0wned!!!%22%29%3C/script%3E%3Cp%20data-foo
```

This would trigger a popup with the string "p0wned!!!".


The Post model (`post.rb`) had a helper method `self.search` that performs an eager-loaded ActiveRecord query using an SQL string with an interpolated variable:

`includes(comments: :replies).where("title like '%#{search}%'")`,

which is now escaped as follows:

`includes(comments: :replies).where([ "title like ?", "%#{search}%"])`

This forces the script embedded in that URL to render on the screen as a harmless string.

A similar escape is now being used in the `PostsController` in the string passed to the call to `logger.info`, just to be safe.

##### XSS
Ivan's blog was also vulnerable to cross-site scripting attacks, like submitting the following string in the search text field:

`foo%' OR true) --`

This would display all posts no matter whether or not they were published.

In `ApplicationController` there is a method `allow_tracking` which is called with a `before_filter`.  This method contained a line:

`response.headers['X-XSS-Protection'] = '0'`

which disabled Rails XSS protection.  I changed the value to `'1'`, but could have just as easily commented out the method and `before_action` call as Rails' default is to enable XSS protection, which turns on XSS auditor to block a page if XSS is detected.

##### secret_token

Ivan's app was exposing it's `secret_token`!!!  This token is used to prevent cookie tampering and commiting it to version control exposes the app to vulnerabilities.  I used the `dotenv-rails` gem to move the token out to a git-ignored `.env` file in the root directory, which is referenced in `secret_token.rb`.

##### Extras

- Just to be safe, I uncommented `config.force_ssl = true` in `config/environments/production.rb`, to force to browser to use HTTPS in production.
- I changed the `content_tag` helpers in the post and search partials into hard-coded html.  These helpers can in some cases be used to compromise apps when they use dynamically interpolated string values, as these did.

### Contributing
1. Fork It
2. Create your feature branch (git checkout -b my-new-feature)
3. Commit your changes (git commit -am 'Add some feature')
4. Push to the branch (git push origin my-new-feature)
5. Create new Pull Request

### Credit
Vulnerabilities exposed in part by the [brakeman](https://github.com/presidentbeef/brakeman) gem

This README was edited at [dillinger.io](dillinger.io)
