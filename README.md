# Sidekiq Cronitor

[Cronitor](https://cronitor.io/) provides dead simple monitoring for cron jobs, daemons, queue workers, websites, APIs, and anything else that can send or receive an HTTP request. The Cronitor Sidekiq library provides a drop in integration for monitoring any Sidekiq Job.


#### NOTE: Version 3.0.0 changes the integration method from 2.x.x. This is a breaking change. You now add the middleware in the Sidekiq initializer. You can opt of out telemetry for specific jobs with a sidekiq_options (see below)

## Installation

Add sidekiq-cronitor your application's Gemfile, near sidekiq:

```ruby
gem 'sidekiq'
gem 'sidekiq-cronitor'
```

And then bundle:

```
bundle
```

## Usage

Configure `sidekiq-cronitor` with an [API Key](https://cronitor.io/docs/api-overview) from [your settings](https://cronitor.io/settings). You can use ENV variables to configure Cronitor:

```sh
export CRONITOR_API_KEY='api_key_123'
export CRONITOR_ENVIRONMENT='development' #default: 'production'

bundle exec sidekiq
```

Or declare the API key directly on the Cronitor module from within your application (e.g. the Sidekiq initializer).

```ruby
require 'cronitor'
Cronitor.api_key = 'api_key_123'
Cronitor.environment = 'development' #default: 'production'
```


To monitor jobs insert the server middleware (most people do this in the Sidekiq initializer)

```ruby
Sidekiq.configure_server do |config|
  config.server_middleware do |chain|
    chain.add Sidekiq::Cronitor::ServerMiddleware
  end
end
```


When this job is invoked, Cronitor will send telemetry pings with a `key` matching the name of your job class (`MyJob` in the example below). If no monitor exists it will create one on the first event. You can configure rules at a later time via the Cronitor dashboard, API, or [YAML config](https://github.com/cronitorio/cronitor-ruby#configuring-monitors) file.

Optional: You can specify the monitor key directly using `sidekiq_options`:

```ruby
class MyJob
  include Sidekiq::Job
  sidekiq_options cronitor_key: 'abc123'

  def perform
  end
end
```


To disable Cronitor for a specific job you can set the following option:

```ruby
class MyJob
  include Sidekiq::Job
  sidekiq_options cronitor_disabled: true

  def perform
  end
end
```

## Disabling For Some Jobs
If you have an entire group or category of jobs you wish to disable monitoring on, it's easiest to create a base class with that option set and then have all your jobs inherit from that base class.

```ruby
  class UnmonitoredJob
    include Sidekiq::Job
    sidekiq_options cronitor_disabled: true
  end

  class NoInstrumentationJob < UnmonitoredJob
    def perform
    end
  end
```

Note: Do NOT set a cronitor_key option on your base class or all your inherited jobs will report under the same job in Cronitor.

## Job Monitoring
If you are using Cronitor to monitor scheduled/periodic jobs and have jobs schedules already defined you can use rake tasks to upload the schedule to Cronitor for monitoring.

In your projects Rakefile you can load the task to be available to you in your project.
It might be a good idea to sync these schedules on every deploy

```ruby
  # your Rakefile, find the path to the gem
  spec = Gem::Specification.find_by_name 'sidekiq-cronitor'
  # if you are using sidekiq_scheduler this task should parse and upload the schedule.
  load "#{spec.gem_dir}/lib/tasks/sidekiq_scheduler.rake"
  # if you are using Sidekiq Pro Periodic Jobs this is an example script
  # Note: this hasn't been tested on Sidekiq Pro yet
  load "#{spec.gem_dir}/lib/tasks/periodic_jobs.rake"
  # You only really need to load one of the rake files unless you are somehow running both systems
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/cronitorio/cronitor-sidekiq/pulls. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
