Tags: Rails

# Rails CDN and Active Storage
When serving assets and images from Rails and ActiveStorage, a CDN is a great way to reduce load on your web server and speed up content delivery to your users.

## Asset CDN
First, even if you're not using ActiveStorage, you'll want to set up a CDN to stand in front of your app. This will proxy requests for images, fonts, javascripts, and stylesheets that live in your app's repository.

The ideal setup here is to serve your assets from your app, set a high cache-expiration, and serve your assets through a CDN. For more information, on serving assets, caching, and cache invalidation, see the [Rails Guides][1].

### The Setup
For a CDN, I recommend Amazon CloudFront. You'll want to create a web distribution which points at the root of your app. Heroku [has a good guide][2] on that.

Once it's set up, set an environment variable (e.g. `ASSET_HOST`) to the distribution domain name and configure your `asset_host`. Also, set a long expiration for your assets. In `config/environments/production.rb`[^1] [^2]:

```ruby
# Tell our CDN and browser to cache assets for a year.
config.public_file_server.headers = { 'Cache-Control' => 'public, max-age=31536000' }

# Serve images, stylesheets, and javascripts from an asset server.
config.action_controller.asset_host = ENV['ASSET_HOST']
```

Once you've configured the above, your production assets should serve from your CloudFront CDN.
---

## ActiveStorage Attachments served through a CDN
When you serve attachments from a cloud storage service (I'm using S3), it will be coming from a different domain than your app. One solution to this would be to [stream the file contents through your app][4].

I opted for creating a second CDN distribution that sits in front of the cloud storage provider that ActiveStorage serves for its 302 redirect[^3]. This does require a little configuration. Thankfully nothing needs to be monkey patched.

1. In `lib/active_storage/service/s3_directory_service.rb`, create the service in [this gist][5].
1. In `config/environment/production.rb`, set ActiveStorage's `service_urls_expire_in` :
	```ruby
	# Allow CDN and Browsers to cache attachments
	config.active_storage.service_urls_expire_in = 1.year
	```
1. In config/storage.yml, configure the service and default `cache_control` for uploaded attachments [by adding the upload key][6] [^4]:
	```yaml
	amazon:
	  service: S3Directory
	  # ...
	  upload:
	    cache_control: 'public, max-age=31536000'
	```
1. When linking to the attachments, use the `rails_blob_url` helper like so: [^5] 
	```ruby
	rails_blob_url user.avatar, host: ENV['ASSET_HOST']
	```
	(For variants, you'll need to call `rails_representation_url` instead.)

[^1]:	If you're using Heroku Review Apps, you'll want to conditionally use `ENV['HEROKU_APP_NAME']` based on `ENV['HEROKU_PARENT_APP_NAME']`.

[^2]:	You may also want to [set your mailers' asset host][3].

[^3]:	See Rails' Representation and Variants controller to see how that works.

[^4]:	If you already have ActiveStorage attachments uploaded in production, you can make them public and add a cache-control header [by using aws cli tools][7].

[^5]:	You may want to create a helper method to make this a little cleaner.

[1]:	https://guides.rubyonrails.org/asset_pipeline.html#cdns-and-the-cache-control-header
[2]:	https://help.heroku.com/8JTD2TJ6/how-should-i-configure-cloudfront-to-work-with-heroku
[3]:	https://guides.rubyonrails.org/action_mailer_basics.html#adding-images-in-action-mailer-views
[4]:	https://github.com/rails/rails/issues/31419#issuecomment-399118697 "on Jun 21, 2018"
[5]:	https://gist.github.com/ericboehs/59ff72b7beeb2724a0979247d0fe7541
[6]:	https://stackoverflow.com/a/58290203
[7]:	https://stackoverflow.com/a/30225271