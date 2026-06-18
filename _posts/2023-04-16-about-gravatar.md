---
title: "About Gravatar"
lang: en
last_modified_at: 2023-04-16T20:00:00-03:00
categories:
  - articles
tags:
  - nonsense
header:
  image: /assets/images/nonsense.jpg
  overlay_image: /assets/images/nonsense.jpg
  overlay_filter: 0.15
  show_overlay_excerpt: false
---

Gravatar, short for "Globally Recognized Avatar," is a service that allows users to create a profile picture that follows them across the web. Launched in 2007 by Tom Preston-Werner, Gravatar quickly became a popular choice for online identity management.

At its core, Gravatar assigns a unique avatar to each user's email address. When users comment on blogs, participate in forums, or interact on various websites, their Gravatar image automatically appears alongside their contributions. This consistent visual representation fosters a sense of identity and community across different online platforms.

One of the key advantages of Gravatar is its convenience. Users can create or update their avatar once, and it will automatically propagate across all Gravatar-enabled sites, eliminating the need to upload a profile picture individually on each platform. This streamlined process saves time and ensures consistency in online branding.

Additionally, Gravatar offers users control over their privacy settings, allowing them to choose between public, private, or rated content for their avatars.

In summary, Gravatar simplifies online identity management, fosters community engagement, and provides users with a convenient and customizable way to represent themselves across the web.

Furthermore, there is no need to create a feature in your application to save your user's photo, we don't need to have storage space for these photos, the user doesn't need to go through the trouble of sending their photo to each application they want to use, and if he doesn't want to be identified, we can still leave a drawing of a funny robot or alien that changes and maintains the same pattern by email.

# How to do it

There are [several Gems](https://rubygems.org/search?query=gravatar) that can do the task. But this is a very easy functionality to have and a simple file added to the /app/models/concerns folder solves the problem. There is no need to [have a gem and have to deal with the hassles of maintaining it](/blog/minimizing-gems-libraries-in-ruby-on-rails/). Especially because many of the gravatar gems I've seen are already very out of date.

So just add the file below:

```ruby
# frozen_string_literal: true

# /app/models/concerns/gravatable.rb

# From:
# https://github.com/chrislloyd/gravtastic/blob/master/lib/gravtastic.rb
# https://chrislloyd.github.io/gravtastic/

require 'digest/md5'

# Module to generate Gravatar URL
module Gravatable
  extend ActiveSupport::Concern

  # The raw MD5 hash of the users' email. Gravatar is particularly tricky as
  # it downcases all emails. This is really the guts of the module,
  # everything else is just convenience.
  def gravatar_id
    Digest::MD5.hexdigest(send(self.class.gravatar_source).to_s.downcase)
  end

  # Constructs the full Gravatar url.
  def gravatar_url(options = {})
    options = {
      rating: 'PG',
      secure: true,
      filetype: :png,
      default: :robohash, # options: 404, mp (mystery person), identicon, monsterid, wavatar, retro, robohash, blank
      size: 200
    }.merge(options)
    gravatar_hostname(options.delete(:secure)) +
      gravatar_filename(options.delete(:filetype)) +
      "?#{url_params_from_hash(self.class.process_options(options))}"
  end

  # private

  # Creates a params hash like "?foo=bar" from a hash like {'foo' => 'bar'}.
  # The values are sorted so it produces deterministic output (and can
  # therefore be tested easily).
  def url_params_from_hash(hash)
    hash.map do |key, val|
      [self.class.gravatar_abbreviations[key.to_sym] || key.to_s, val.to_s].join('=')
    end.sort.join('&')
  end

  # Returns either Gravatar's secure hostname or not.
  def gravatar_hostname(secure)
    "http#{secure ? 's://secure.' : '://'}gravatar.com/avatar/"
  end

  # Munges the ID and the filetype into one. Like "abc123.png"
  def gravatar_filename(filetype)
    "#{gravatar_id}.#{filetype}"
  end

  # Methods for the Model Class
  module ClassMethods
    attr_accessor :gravatar_source

    def gravatar_field(source = :emailx)
      @gravatar_source = source
    end

    # Some options need to be processed before becoming URL params
    def process_options(options)
      processed_options = {}
      options.each do |key, val|
        case key
        when :forcedefault
          processed_options[key] = 'y' if val
        else
          processed_options[key] = val
        end
      end
      processed_options
    end

    def gravatar_abbreviations
      { size: 's',
        default: 'd',
        rating: 'r',
        forcedefault: 'f' }
    end
  end
end
```

In any model that has the email field, just add the Gravatable as below:

```ruby
class User
  include Gravatable

  gravatar_field :email
end

User.last.gravatar_url({ size: 40 })
```

As you can see in the code, it is not mine originally. I created a version from the [Gravtastic gem](https://github.com/chrislloyd/gravtastic/commits/master/) that has not been updated since 2016.
