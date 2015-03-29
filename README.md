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

## question methods

Bacon allows you to test the truthiness of a `x?` method, such as `.kind_of?`. Remove the question mark from the method to test it, like `x.should.be.kind_of(Hash)`.

```ruby
describe Hash do
  it "is a hash instance" do
    obj = {}
    obj.should.be.kind_of(Hash)
  end
end
```

## be, a, an

These are mainly just syntactic sugar so you can write something like this: 

```ruby
describe Hash do
  it "is a hash instance" do
    obj = {}
    obj.should.be.a.kind_of(Hash)
  end
end
```

## not

Tests that the opposite is true.

```ruby
describe Hash do
  it "is not an array" do
    obj = {}
    obj.should.not.be.kind_of(Array)
  end
end
```

## Exceptions

```ruby
describe "Viper::SnakeCase" do
  it "has a 'Viper::SnakeCase' module" do
    should.not.raise(NameError) { Viper::SnakeCase }
  end
end
```

## tests MyViewController

You can mount a UIViewController into the simulator with the `tests` class method. This will look for a `controller` method (or provide its own if you don't).

```ruby

```

## wait_till/wait_max

Keeps trying the block until it returns a truthy value, up to the timeout specified (defaulted to 3 seconds).

```ruby
describe "HTTP call" do
  it "returns a result" do
    @ip = nil
    AFMotion::JSON.get("http://ip.jsontest.com/") do |result|
      @ip = result.object["ip"]
    end
    wait_till 20 { @ip.nil? == false }
    @ip.should == "12.34.56.78"
  end
end
```

Another way to approach this:

```ruby
describe "HTTP call" do
  it "returns a result" do
    @ip = nil
    AFMotion::JSON.get("http://ip.jsontest.com/") do |result|
      @ip = result.object["ip"]
      resume
    end
    wait_max 20 do
      @ip.should == "12.34.56.78"
    end
  end
end
```

## Useful Gems

### [motion-juxtapose](https://github.com/terriblelabs/motion-juxtapose)

Visual regression testing. You get a *lot* of value with a small test.

```ruby
gem "motion-juxtapose"
```

```ruby
describe SettingsScreen do
  tests SettingsScreen

  it "looks like a SettingsScreen" do
    views(UIView).length.should.be > 0 # Ensure views are loaded first
    it_should_look_like "SettingsScreen", 4 # 4% "fuzz factor"
  end
end
```

### [motion-stump](https://github.com/siuying/motion-stump/)

```ruby
gem "motion-stump"
```

```ruby
describe "
```



## Debugging tests

Sometimes, you'll get very useless output from a failed test. Try changing your spec output style.

```ruby
# In your Rakefile
ENV["output"] ||= "tap"
```









