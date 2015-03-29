# RubyMotion Bacon Specs Cheat Sheet

by Jamon Holmgren

RubyMotion ships with a built-in fork of [MacBacon](https://github.com/alloy/MacBacon), which is itself a fork of [Bacon](https://github.com/chneukirchen/bacon), a small pure-Ruby RSpec clone.

RubyMotion's Bacon is fairly capable but not all that well documented. This is a cheat sheet to help bring to mind various strategies for testing RubyMotion apps and gems.

## describe/context blocks

`describe` and `context` are [literally aliases](https://github.com/HipByte/RubyMotion/blob/master/lib/motion/spec.rb#L649), but you usually use `describe` to provide a description for the overall goal of a test, and `context` to establish various scenarios you're running your tests.

```ruby
describe "String description" do
  # ...
end

describe UIViewController do
  # ...
end
```

You can provide `before` and `after` blocks in any describe/context block.

```ruby
describe PM::TableScreen do
  context "with a nav_bar" do
    before do
      @screen = PM::TableScreen.new(nav_bar: true)
    end
    after do
      @screen = nil
    end
    
    it "has a nav_bar" do
      @screen.navigationController.should.be.kind_of(PM::NavigationController)
    end
  end
end
```

## should

The basic assertation method of Bacon.

```ruby
describe Hash do
  it "is a hash instance" do
    obj = {}
    obj.should == {}
  end
end
```

## be

`.be` allows you to test the truthiness of a `x?` method, such as `.kind_of?`. Remove the question mark from the method, like `x.should.be.kind_of(Hash)`.

```ruby
describe Hash do
  it "is a hash instance" do
    obj = {}
    obj.should.be.kind_of(Hash)
  end
end
```





