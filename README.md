# [with_model](https://github.com/Casecommons/with_model)

[![Build Status](https://secure.travis-ci.org/Casecommons/with_model.png?branch=master)](https://travis-ci.org/Casecommons/with_model)
[![Code Climate](https://codeclimate.com/github/Casecommons/with_model.png)](https://codeclimate.com/github/Casecommons/with_model)
[![Coverage Status](https://coveralls.io/repos/Casecommons/with_model/badge.png?branch=master)](https://coveralls.io/r/Casecommons/with_model)
[![Gem Version](https://badge.fury.io/rb/with_model.png)](https://rubygems.org/gems/with_model)
[![Dependency Status](https://gemnasium.com/Casecommons/with_model.png)](https://gemnasium.com/Casecommons/with_model)

`with_model` dynamically builds an ActiveRecord model (with table) before each test in a group and destroys it afterwards.

## Installation

Install as usual: `gem install with_model` or add `gem 'with_model'` to your Gemfile. See `.travis.yml` for supported (tested) Ruby versions.

## RSpec

Extend `WithModel` into RSpec:

```ruby
require 'with_model'

RSpec.configure do |config|
  config.extend WithModel
end
```

## minitest/spec

Extend `WithModel` into minitest/spec:

```ruby
require 'with_model'

class Minitest::Spec
  extend WithModel
end
```

## Usage

After setting up as above, call `with_model` and inside its block pass it a `table` block and a `model` block.

```ruby
require 'spec_helper'

describe "A blog post" do
  before :all do
    module SomeModule; end
  end

  after :all do
    Object.send :remove_const, :SomeModule
  end

  with_model :BlogPost do
    # The table block works just like a migration.
    table do |t|
      t.string :title
      t.timestamps
    end

    # The model block works just like the class definition.
    model do
      include SomeModule
      has_many :comments
      validates_presence_of :title

      def self.some_class_method
        'chunky'
      end

      def some_instance_method
        'bacon'
      end
    end
  end

  # with_model classes can have associations.
  with_model :Comment do
    table do |t|
      t.string :text
      t.belongs_to :blog_post
      t.timestamps
    end

    model do
      belongs_to :blog_post
    end
  end

  it "can be accessed as a constant" do
    expect(BlogPost).to be
  end

  it "has the module" do
    expect(BlogPost.include?(SomeModule)).to be_true
  end

  it "has the class method" do
    expect(BlogPost.some_class_method).to eq 'chunky'
  end

  it "has the instance method" do
    expect(BlogPost.new.some_instance_method).to eq 'bacon'
  end

  it "can do all the things a regular model can" do
    record = BlogPost.new
    expect(record).to_not be_valid
    record.title = "foo"
    expect(record).to be_valid
    expect(record.save).to be_true
    expect(record.reload).to eq record
    record.comments.create!(:text => "Lorem ipsum")
    expect(record.comments.count).to eq 1
  end

  # with_model classes can have inheritance.
  class Car < ActiveRecord::Base
    self.abstract_class = true
  end

  with_model :Ford, superclass: Car do
  end

  it "has a specified superclass" do
    expect(Ford < Car).to be_true
  end
end

describe "another example group" do
  it "does not have the constant anymore" do
    expect(defined?(BlogPost)).to be_false
  end
end

describe "with table options" do
  with_model :WithOptions do
    table :id => false do |t|
      t.string 'foo'
      t.timestamps
    end
  end

  it "respects the additional options" do
    expect(WithOptions.columns.map(&:name)).to_not include("id")
  end
end
```

## Requirements

- Ruby 1.9.3+
- RSpec or minitest/spec
- ActiveRecord 3+ (for ActiveRecord 2, use with_model 0.2.x)

## Versioning

As of version 1.0.0, with_model uses [Semantic Versioning 2.0.0](http://semver.org/spec/v2.0.0.html). Previous versions used an ad-hoc versioning scheme. 

## License

Copyright © 2010–2014 Case Commons, LLC.
Licensed under the MIT license, available in the “LICENSE” file.
