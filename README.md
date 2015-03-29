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
describe MyScreen do
  tests MyScreen
  
  def controller
    @controller ||= MyScreen.new
  end
  
  after { @controller = nil }
  
  it "has the right title" do
    view("My Screen").should.be.kind_of(UILabel)
  end
end
```

If you want to have your screen in a navigation controller, make sure your `controller` method returns the navigationController.

```ruby
describe MyScreen do
  tests MyScreen
  
  def screen
    @screen ||= MyScreen.new(nav_bar: true) # ProMotion-style
  end
  
  def controller
    screen.navigationController
  end
  
  after { @screen = nil }
  
  it "has the right title" do
    view("My Screen").should.be.kind_of(UILabel)
  end
end
```

## tap

Taps a button on the screen.

```ruby
describe MyScreen do
  tests MyScreen
  
  def controller
    @controller ||= MyScreen.new
  end
  
  after { @controller = nil }

  it "has a button" do
    tap("Go Forth And Conquer")
    view("Conquered!").should.be.present
  end
end
```

## Testing HTTP requests

Use `wait_till` which keeps trying the block until it returns a truthy value, up to the timeout specified (defaulted to 3 seconds).

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

Another way to approach this is to use `wait_max` and the `resume` command, which is what I recommend:

```ruby
describe "HTTP call" do
  it "returns a result" do
    @ip = nil
    AFMotion::JSON.get("http://ip.jsontest.com/") do |result|
      result.should.be.success
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

`.stub!` will replace a method and return the result you specify. It doesn't care if it's called or not, though.

```ruby
  screen = MyScreen.new(arg: true)
  screen.stub!(:foos, return: [])
  screen.stub!(:bars, return: [ {}, {} ])
```

`.mock!` is the same as `.stub!`, but will fail the test if it's not called.

You can also pass a block to do more stuff, including assertations:

```ruby
it "does a Google search" do
  API::Client.mock!(:get) do |url, params|
    url.should == "http://google.com"
    params[:q].should == "Nickelback sucks"
    resume
  end
  wait_max 20 {}
end
```

### [motion-facon](https://github.com/svyatogor/motion-facon)

A good alternative to motion-stump (above). I haven't used it all that much, but its syntax is very pretty. It's fallen a bit out of date, however.

```ruby
  describe 'PersonController' do
    extend Facon::SpecHelpers

    before do
      @konata = mock('konata', :id => 1, :name => 'Konata Izumi')
      @kagami = mock('kagami', :id => 2, :name => 'Kagami Hiiragi')
    end

    it "should find all people" do
      Person.should.receive(:find).with(:all).and_return([@konata, @kagami])

      Person.find(:all).should == [@konata, @kagami]
    end
  end
```

## Debugging tests

Sometimes, you'll get very useless output from a failed test. Try changing your spec output style.

```ruby
# In your Rakefile
ENV["output"] ||= "tap"
```









