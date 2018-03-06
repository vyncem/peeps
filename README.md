# README

Peeps: A demo of JSONAPI-Resources:  https://github.com/cerebris/peeps

# setup
`bin/rails new peeps -d postgresql --skip-javascript`

`rake db:create`

`cat gem 'jsonapi-resources' > Gemfile`

`bundle`

# Application Controller
```
class ApplicationController < ActionController::Base
  include JSONAPI::ActsAsResourceController

  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :null_session
end
```

# environment
`config/environments/development.rb`

```
# Eager load code on boot so JSONAPI-Resources resources are loaded and processed globally
config.eager_load = true

# Prevent returning HTML formatted error messages on exception. makes for nicer output when debugging using curl or a client library.
config.consider_all_requests_local       = false
```

# models
`bin/rails g model Contact name_first:string name_last:string email:string twitter:string`

```
class Contact < ActiveRecord::Base
  has_many :phone_numbers

  ### Validations
  validates :name_first, presence: true
  validates :name_last, presence: true

end
```

`bin/rails g model PhoneNumber contact_id:integer name:string phone_number:string`

```
class PhoneNumber < ActiveRecord::Base
  belongs_to :contact
end
```

# Migrate DB
`bin/rails db:migrate`

# controllers
```
bin/rails g controller Contacts --skip-assets
bin/rails g controller PhoneNumbers --skip-assets
```

# resources
`mkdir app/resources`

contact_resource.rb
```
class ContactResource < JSONAPI::Resource
  attributes :name_first, :name_last, :email, :twitter
  has_many :phone_numbers
end
```

and phone_number_resource.rb
```
class PhoneNumberResource < JSONAPI::Resource
  attributes :name, :phone_number
  has_one :contact

  filter :contact
end
```

# routes
```
jsonapi_resources :contacts
jsonapi_resources :phone_numbers
```

# Test

- start server
`bin/rails s`
- create contact
`curl -i -H "Accept: application/vnd.api+json" -H 'Content-Type:application/vnd.api+json' -X POST -d '{"data": {"type":"contacts", "attributes":{"name-first":"John", "name-last":"Doe", "email":"john.doe@boring.test"}}}' http://localhost:3000/contacts`
- create phone number
`curl -i -H "Accept: application/vnd.api+json" -H 'Content-Type:application/vnd.api+json' -X POST -d '{ "data": { "type": "phone-numbers", "relationships": { "contact": { "data": { "type": "contacts", "id": "1" } } }, "attributes": { "name": "home", "phone-number": "(603) 555-1212" } } }'` http://localhost:3000/phone-numbers
- list contacts
`curl -i -H "Accept: application/vnd.api+json" "http://localhost:3000/contacts"`
- list contacts including phone numbers
`curl -i -H "Accept: application/vnd.api+json" "http://localhost:3000/contacts?include=phone-numbers"`
- list contacts including phone numbers and some fields
`curl -i -H "Accept: application/vnd.api+json" "http://localhost:3000/contacts?include=phone-numbers&fields%5Bcontacts%5D=name-first,name-last&fields%5Bphone-numbers%5D=name"`
- invalid contact creation
`curl -i -H "Accept: application/vnd.api+json" -H 'Content-Type:application/vnd.api+json' -X POST -d '{ "data": { "type": "contacts", "attributes": { "name-first": "John Doe", "email": "john.doe@boring.test" } } }' http://localhost:3000/contacts`
