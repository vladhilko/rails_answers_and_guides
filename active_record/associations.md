1. How to allow `nil` value for the `belongs_to` association?

```rb
  class Book < ActiveRecord::Base
    belongs_to :author, optional: true
  end
```
