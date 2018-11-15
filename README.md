# BrowseEverything

Code:
[![Gem Version](https://badge.fury.io/rb/browse-everything.png)](http://badge.fury.io/rb/browse-everything)
[![Build Status](https://travis-ci.org/samvera/browse-everything.svg?branch=master)](https://travis-ci.org/samvera/browse-everything)
[![Coverage Status](https://coveralls.io/repos/samvera/browse-everything/badge.svg?branch=master&service=github)](https://coveralls.io/github/samvera/browse-everything?branch=master)

Docs:
[![Contribution Guidelines](http://img.shields.io/badge/CONTRIBUTING-Guidelines-blue.svg)](./CONTRIBUTING.md)
[![Apache 2.0 License](http://img.shields.io/badge/APACHE2-license-blue.svg)](./LICENSE.txt)

Jump in: [![Slack Status](http://slack.samvera.org/badge.svg)](http://slack.samvera.org/)

# FORK FOR BOOTSTRAP4

See https://github.com/samvera/browse-everything/issues/246 (

An experimental and unsustainable (no promise of maintenance) fork of browse-everything to get it working with bootstrap 4.

* Removed bootstrap-sass (bootstrap3) dependency -- you need to include `bootstrap` gem in your app yourself.
* removed fontawesome-rails dependency, I couldn't figure out how/if it was being used.
* removed (BS3) 'content-columns' mixin from scss
* Completely rewrote CSS to look right with bootstrap4, also possibly improving some things
that could have looked better previously too.

Removed the intermediate browse_everything.scss file that then imported other files -- it really complicated the scss in weird hard to debug ways, for unclear purpose. This also potentially paves the way for a browse-everything that supports bootstrap 3 or 4, you could just @import different scss files in your main application.scss -- and they'll have access to either bootstrap 3 or 4 variables/mixins, that you already imported yourself.

## Changed installation directions

1. Run `rails g browse_everything:install` to insert into your routes.yml and create a browse-everything.yml. (No longer generates an scss asset. Had long ago stopped generating javascript, despite generator description)
2. Your app should have the bootstrap4 `bootstrap` gem as a dependency itself, and should have an
application.scss that already has `@import 'bootstrap'` in it.
3. Add `@import 'browse_everything'` after the bootstrap import.
4. Add to your JS `//= require browse_everything` (also jquery if not already there)

The previous install process had you have to do 3 and 4 _anyway_ -- the intermediary .scss file was not helping. We could conceivably write a generator to modify your application.js and application.scss, don't know if it's worth it.


# What is BrowseEverything?

This Gem allows your rails application to access user files from cloud storage.
Currently there are drivers implemented for [Dropbox](http://www.dropbox.com),
[Google Drive](http://drive.google.com),
[Box](http://www.box.com), [Amazon S3](https://aws.amazon.com/s3/),
and a server-side directory share.

The gem uses [OAuth](http://oauth.net/) to connect to a user's account and
generate a list of single use urls that your application can then use to
download the files.

**This gem does not depend on hydra-head**

## Product Owner & Maintenance

BrowseEverything is a Core Component of the Samvera community. The documentation for
what this means can be found
[here](http://samvera.github.io/core_components.html#requirements-for-a-core-component).

### Product Owner

[mbklein](https://github.com/mbklein)

# Getting Started

## Supported Ruby Releases
Currently, the following releases of Ruby are supported:
- 2.5.1
- 2.4.4
- 2.3.7
  - Please note that this is the last release in the 2.3.x series, and support is [scheduled to be withdrawn](https://www.ruby-lang.org/en/news/2018/03/28/ruby-2-3-7-released/).  We would strongly recommend that one upgrades from 2.3.7 in response to this.

## Supported Rails Releases
- 5.2.0
- 5.1.6
- 5.0.7
- 4.2.10
  - The supported Rail releases follow those specified by [the security policy of the Rails Community](https://rubyonrails.org/security/).  As is the case with the supported Ruby releases, it is recommended that one upgrades from any Rails release no longer receiving security updates.

## Installation

Add this lines to your application's Gemfile:

    gem 'jquery-rails'
    gem 'browse-everything'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install browse-everything

### Configuring the gem

After installing the gem, run the generator

    $ rails g browse_everything:install

This generator will set up the _config/browse_everything_providers.yml_ file and add the browse-everything engine to your application's routes.

If you prefer not to use the generator, or need info on how to set up providers in the browse_everything_providers.yml, use the info on [Configuring browse-everything](https://github.com/samvera/browse-everything/wiki/Configuring-browse-everything).

### Include the CSS and JavaScript

Add `@import "browse_everything";` to your application.css.scss

In `app/assets/javascripts/application.js` include jquery and the BrowseEverything

```javascript
//= require jquery
//= require browse_everything
```

## Testing
This is a Rails Engine which is tested using the [engine_cart](https://github.com/cbeer/engine_cart) Gem.  Test suites may be executed with the following invocation:

```bash
bundle exec rake
```

### Testing Problems
Should you attempt to execute the test suite and encounter the following error:
```bash
Your Ruby version is 2.x.x, but your Gemfile specified 2.y.z
```
...then you must clean the internal test app generated by `engine_cart` with the following:
```bash
bundle exec rake engine_cart:clean
```

## Usage

### Adding Providers
In order to connect to a provider like [Dropbox](http://www.dropbox.com),
[Google Drive](http://drive.google.com), or
[Box](http://www.box.com), you must provide API keys in _config/browse_everything_providers.yml_.  For info on how to edit this file, see [Configuring browse-everything](https://github.com/samvera/browse-everything/wiki/Configuring-browse-everything)

### Views

browse-everything can be triggered in two ways -- either via data attributes in an HTML tag or via JavaScript.  Either way, it accepts the same options:

#### Options

| Name            | Type            | Default         | Description |
|-----------------|-----------------|-----------------|----------------------------------------------------------------|
| route           | path (required) | ''              | The base route of the browse-everything engine.                |
| target          | xpath or jQuery | null            | A form object to add the results to as hidden fields.          |
| context         | text            | null            | App-specific context information (passed with each request)    |
| accept          | MIME mask       | */*             | A list of acceptable MIME types to browse (e.g., 'video/*')    |

If a `target` is provided, browse-everything will automatically convert the JSON response to a series of hidden form fields
that can be posted back to Rails to re-create the array on the server side.

#### Via data attributes

To trigger browse-everything using data attributes, set the _data-toggle_ attribute to "browse-everything" on the HTML tag.  This tells the javascript where to attach the browse-everything behaviors. Pass in the options using the _data-route_ and _data-target_ attributes, as in `data-target="#myForm"`.

For example:

```html
<button type="button" data-toggle="browse-everything" data-route="<%=browse_everything_engine.root_path%>"
  data-target="#myForm" class="btn btn-large btn-success" id="browse">Browse!</button>
```

#### Via JavaScript

To trigger browse-everything via javascript, use the .browseEverything() method to attach the behaviors to DOM elements.

```javascript
$('#browse').browseEverything(options)
```

The options argument should be a JSON object with the route and (optionally) target values set.  For example:
```javascript
$('#browse').browseEverything({
  route: "/browse",
  target: "#myForm"
})
```

See [JavaScript Methods](https://github.com/samvera/browse-everything/wiki/JavaScript-Methods) for more info on using javascript to trigger browse-everything.

### The Results (Data Structure)

browse-everything returns a JSON data structure consisting of an array of URL specifications. Each URL specification
is a plain object with the following properties:

| Property           | Description |
|--------------------|--------------------------------------------------------------------------------------|
| url                | The URL of the selected remote file. |
| auth_header        | Any headers that need to be added to the request in order to access the remote file. |
| expires            | The expiration date/time of the specified URL. |
| file_name          | The base name (filename.ext) of the selected file.                                   |

For example, after picking two files from dropbox,

If you initialized browse-everything via JavaScript, the results data passed to the `.done()` callback will look like this:
```json
[
  {
    "url": "https://dl.dropbox.com/fake/filepicker-demo.txt.txt",
    "expires": "2014-03-31T20:37:36.214Z",
    "file_name": "filepicker-demo.txt.txt"
  }, {
    "url": "https://dl.dropbox.com/fake/Getting%20Started.pdf",
    "expires": "2014-03-31T20:37:36.731Z",
    "file_name": "Getting Started.pdf"
  }
]
```
See [JavaScript Methods](https://github.com/samvera/browse-everything/wiki/JavaScript-Methods) for more info on using javascript to trigger browse-everything.

If you initialized browse-everything via data-attributes and set the _target_ option (via the _data-target_ attribute or via the _target_ option on the javascript method), the results data be written as hidden fields in the `<form>` you've specified as the target.  When the user submits that form, the results will look like this:
```ruby
"selected_files" => {
  "0"=>{
    "url"=>"https://dl.dropbox.com/fake/filepicker-demo.txt.txt",
    "expires"=>"2014-03-31T20:37:36.214Z",
    "file_name"=>"filepicker-demo.txt.txt"
  },
  "1"=>{
    "url"=>"https://dl.dropbox.com/fake/Getting%20Started.pdf",
    "expires"=>"2014-03-31T20:37:36.731Z",
    "file_name"=>"Getting Started.pdf"
  }
}
```

### Retrieving Files

The `BrowseEverything::Retriever` class has two methods, `#retrieve` and `#download`, that
can be used to retrieve selected content. `#retrieve` streams the file by yielding it, chunk
by chunk, to a block, while `#download` saves it to a local file.

Given the above response data:

```ruby
retriever = BrowseEverything::Retriever.new
download_spec = params['selected_files']['1']

# Retrieve the file, yielding each chunk to a block
retriever.retrieve(download_spec) do |chunk, retrieved, total|
  # do something with the `chunk` of data received, and/or
  # display some progress using `retrieved` and `total` bytes.
end

# Download the file. If `target_file` isn't specified, the
# retriever will create a tempfile and return the name.
retriever.download(download_spec, target_file) do |filename, retrieved, total|
  # The block is still useful for showing progress, but the
  # first argument is the filename instead of a chunk of data.
end
```

### Examples

See `spec/support/app/views/file_handler/index.html` for an example use case. You can also run `rake app:generate` to
create a fully-functioning demo app in `spec/internal` (though you will have to create
`spec/internal/config/browse_everything.providers.yml` file with your own configuration info.)

# Help

The Samvera community is here to help. Please see our [support guide](./SUPPORT.md).

# Acknowledgments

This software has been developed by and is brought to you by the Samvera community.  Learn more at the
[Samvera website](http://samvera.org/).

![Samvera Logo](https://wiki.duraspace.org/download/thumbnails/87459292/samvera-fall-font2-200w.png?version=1&modificationDate=1498550535816&api=v2)
