[![Build Status](https://travis-ci.org/iaintshine/ruby-tracing-matchers.svg?branch=master)](https://travis-ci.org/iaintshine/ruby-tracing-matchers)

# Tracing::Matchers

The gem exposes RSpec-compatible convenient one-liners, matchers, which you can use to test OpenTracing instrumentations, without having to use production OpenTracing Tracer e.g. Jaeger. It's expected that the gem will be used together with [test-tracer](https://github.com/iaintshine/ruby-test-tracer) - an OpenTracing compatible test Tracer implementation.

## Documentation

For far more detailed documentation and usage examples check [matchers.rb](https://github.com/iaintshine/ruby-tracing-matchers/blob/master/lib/tracing/matchers.rb), [span_matchers.rb](https://github.com/iaintshine/ruby-tracing-matchers/blob/master/lib/tracing/matchers/span_matchers.rb), [specs](https://github.com/iaintshine/ruby-tracing-matchers/tree/master/spec/tracing) and [rubydoc.info](http://www.rubydoc.info/github/iaintshine/ruby-tracing-matchers/master/Tracing/Matchers).

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'tracing-matchers'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install tracing-matchers

## Available matchers

```ruby
expect(tracer).to have_traces(n)
                  .started
                  .finished
expect(tracer).to have_spans(n)
                  .started
                  .finished
expect(tracer).to have_span(operation_name)
                  .in_progress
                  .started
                  .finished
                  .with_tags(*tags)
                  .with_logs(**fields)
                  .with_log(**fields)
                  .with_baggage(*baggage)
                  .child_of(operation_name|span|span_context)
                  .following_after(operation_name|span)

expect(span).to have_tag(*tags)
expect(span).to have_tags(*tags)
expect(span).to have_log(**fields)
expect(span).to have_logs(**fields)
expect(span).to have_baggage(*baggage)
expect(span).to have_baggage_item(*baggage)
expect(span).to have_parent
expect(span).to be_child_of(opration_name|span|span_context)
expect(span).to follow_after(operation_name|span)
```

## Usage

High-level test examples.

```ruby
require 'tracing-matchers'

describe "traced code" do
  context "when we expect no traces" do
    it "does not have any traces" do
      expect(tracer).not_to have_traces
    end
  end

  context "when we expect traces to be present" do
    it "does have some traces started" do
      expect(tracer).to have_traces
    end
  end

  context "when we expect exactly N traces" do
    it "has N traces recorded" do
      expect(tracer).to have_traces(N)
    end
  end
end
```

```ruby
describe "traced code" do
  context "when we expect no spans" do
    it "does not have any spans" do
      expect(tracer).not_to have_spans
    end
  end

  context "when we expect spans to be present" do
    it "does have some spans started" do
      expect(tracer).to have_spans
    end
  end

  context "when we expect exactly N spans" do
    it "has N spans recorded" do
      expect(tracer).to have_spans(N)
    end
  end
end
```

Span-level test examples.

```ruby
describe "traced code" do
  context "when a new span was started" do
    it "is in progress" do
      expect(tracer).to have_span.in_progress
    end

    it "does have the proper name" do
      expect(tracer).to have_span("operation name")
    end

    it "does include standard OT tags" do
      expect(tracer).to have_span.with_tags('component' => 'ActiveRecord')
      expect(span).to have_tag('component', 'ActiveRecord')
    end

    it "does include a baggage item" do
      expect(tracer).to have_span.with_baggage('user_id', '1')
      expect(span).to have_baggage('user_id', '1')
    end

    it "does not have any log entries" do
      expect(tracer).not_to have_span.with_logs
    end
  end

  context "when an event was logged" do
    it "does have some log entries recorded" do
      expect(tracer).to have_span.with_logs
      expect(span).to have_log 
    end

    it "includes all the event attributes" do
      expect(tracer).to have_span.with_log(event: "exceptional message", severity: Logger::ERROR)
      expect(span).to have_log(event: "exceptional message", severity: Logger::ERROR)
    end
  end

  context "when a span was finished" do
    it "is not in progress" do
      expect(tracer).to have_span.finished 
    end
  end
end
```

Span context-level test examples.

```ruby
describe "traced code" do
  context "when a new span was started as child" do
    it "has a prent" do
      expect(tracer).to have_span("child span").child_of("root span")
      expect(span).to be_child_of("root span")
    end
  end
end
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/iaintshine/ruby-tracing-matchers. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.
