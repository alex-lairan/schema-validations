# Schemas

Schemas come to solve a simple problem. Sometimes we would like to have type-safe guarantee params when parsing HTTP parameters or Hash(String, String) for a request moreover; Schemas are to resolve precisely this problem with the added benefit of performing
business rules validation to have the params adhere to a `"business schema"`.

Schemas are beneficial, in my opinion, ideal, for when defining web services APIs

## Installation

Add this to your application's `shard.yml`:

```yaml
dependencies:
  schemas:
    github: your-github-user/schemas
```

## Usage

```crystal
require "schemas"
```

```crystal
class ExampleController
  getter params : Hash(String, String)

  def initialize(@params)
  end

  schema("User") do
    param email : String, match: /\w+@\w+\.\w{2,3}/, message: "Email must be valid!"
    param name : String, size: (1..20)
    param age : Int32, gte: 24, lte: 25, message: "Must be 24 and 30 years old"
    param alive : Bool, eq: true
    param childrens : Array(String)
    param childrens_ages : Array(Int32)

    schema("Address") do
      param street : String, size: (5..15)
      param zip : String, match: /\d{5}/
      param city : String, size: 2, in: %w[NY NJ CA UT]
    end
  end
end

params = HTTP::Params.parse(
  "email=test@example.com&name=john&age=24&alive=true&" +
  "childrens=Child1,Child2&childrens_ages=1,2&" +
  // Nested Params
  "address.city=NY&address.street=Sleepy Hollow&address.zip=12345"
)

subject = ExampleController.new(params.to_h)
```

Schemas are defined as value objects, meaning structs, which are NOT mutable,
making them ideal to pass schema object as arguments to constructors.

```crystal
user = subject.user
address = user.address

user.valid?.should be_true
address.valid?.should be_true
```

### Casting to Custom Types

This is WIP

```crystal
class CustomType
  include Schema::CastAs(CustomType)

  def initialize(@value : String)
  end

  def value
    convert(self.class)
  end

  // Provide a convert method to handle the casting
  def convert(asType : self.class)
    @value.split(",").map { |i| i.to_i32 }
  end
end
```

### Custom Validations

This is WIP.

Create a module that extends from schema Validators module.

```crystal
require "some_model"

module Validators
  module Custom
    def unique?(value, enabled : Bool = true)
      SomeModel.where(name: value).count.zero? if enabled
    end
  end
end

schema("User") do
  param name : String, unique: true
end
```

Notice that `unique:` corresponds to `unique?`.
This is how the library know which validation to perform.

## Development

> Note: This is subject to modifications for improvement.
> Submit ideas as issues before opening a pull request.


## Contributing

1. Fork it (<https://github.com/your-github-user/schemas/fork>)
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Contributors

- [your-github-user](https://github.com/your-github-user) Elias J. Perez - creator, maintainer
