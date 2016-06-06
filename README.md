# HTML5 File uploader for rails

This gem use https://github.com/blueimp/jQuery-File-Upload for upload files.

## Install

In Gemfile:

```
gem 'rails-uploader'
```

In routes:

``` ruby
mount Uploader::Engine => '/uploader'
```

Migration for ActiveRecord:

```bash
$> rake uploader:install:migrations
```

## Usage

Architecture to store uploaded files (cancan integration):

``` ruby
class Asset < ActiveRecord::Base
  include Uploader::Asset
end

class Picture < Asset
  mount_uploader :data, PictureUploader, mount_on: :data_file_name
  validates :data, file_size: { maximum: 5.megabytes.to_i }

  def thumb_url
    url(:thumb)
  end
end
```

For example user has one picture:

``` ruby
class User < ActiveRecord::Base
  has_one :picture, as: :assetable, dependent: :destroy

  fileuploads :picture
end
```

Find asset by foreign key or guid:

``` ruby
@user.fileupload_asset(:picture)
```

### Include assets

Javascripts:

```
//= require uploader/application
```

Stylesheets:

```
*= require uploader/application
```

### Views

```erb
<%= uploader_field_tag :article, :photo %>
```

or FormBuilder:

```erb
<%= form.uploader_field :photo, sortable: true %>
```

### Formtastic

```erb
<%= f.input :pictures, :as => :uploader %>
```

### SimpleForm

```erb
<%= f.input :pictures, :as => :uploader, :input_html => {:sortable => true} %>
```

#### Confirming deletions

This is only working in Formtastic and FormBuilder:

``` erb
# formtastic
<%= f.input :picture, :as => :uploader, :confirm_delete => true %>
# the i18n lookup key would be en.formtastic.delete_confirmations.picture
```

## Authorization

Setup custom authorization adapter and current user:

``` ruby
# config/initializers/uploader.rb
Uploader.setup do |config|
  config.authorization_adapter = CanCanUploaderAdapter
  config.current_user_proc = -> (request) { request.env['warden'].user }
end
```

``` ruby
class CanCanUploaderAdapter < Uploader::AuthorizationAdapter
  def authorized?(action, subject = nil)
    cancan_ability.can?(action, subject)
  end

  def scope_collection(collection, action = :index)
    collection.accessible_by(cancan_ability, action)
  end

  protected

  def cancan_ability
    @cancan_ability ||= Ability.new(user)
  end
end

## JSON Response

https://github.com/blueimp/jQuery-File-Upload/wiki/Setup#using-jquery-file-upload-ui-version-with-a-custom-server-side-upload-handler

Extend your custom server-side upload handler to return a JSON response akin to the following output:

``` json
{"files": [
  {
    "name": "picture1.jpg",
    "size": 902604,
    "url": "http:\/\/example.org\/files\/picture1.jpg",
    "thumbnailUrl": "http:\/\/example.org\/files\/thumbnail\/picture1.jpg",
    "deleteUrl": "http:\/\/example.org\/files\/picture1.jpg",
    "deleteType": "DELETE"
  },
  {
    "name": "picture2.jpg",
    "size": 841946,
    "url": "http:\/\/example.org\/files\/picture2.jpg",
    "thumbnailUrl": "http:\/\/example.org\/files\/thumbnail\/picture2.jpg",
    "deleteUrl": "http:\/\/example.org\/files\/picture2.jpg",
    "deleteType": "DELETE"
  }
]}
```

To return errors to the UI, just add an error property to the individual file objects:

``` json
{"files": [
  {
    "name": "picture1.jpg",
    "size": 902604,
    "error": "Filetype not allowed"
  },
  {
    "name": "picture2.jpg",
    "size": 841946,
    "error": "Filetype not allowed"
  }
]}
```

When removing files using the delete button, the response should be like this:

``` json
{"files": [
  {
    "picture1.jpg": true
  },
  {
    "picture2.jpg": true
  }
]}
```

Note that the response should always be a JSON object containing a files array even if only one file is uploaded.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

Copyright (c) 2013 Fodojo, released under the MIT license
