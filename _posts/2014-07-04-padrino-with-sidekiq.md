---
layout: post
title: "Padrino with Sidekiq"
tags:
- ruby
date: 2014-07-30 14:30:00
---

I recently added [sidekiq](https://github.com/mperham/sidekiq) to a
[padrino](https://github.com/padrino/padrino-framework) app. Mounting
the sinatra app wasn't completely straightforward, and I didn't find
everything I needed in one place. So here's an attempt to do just
that.

First, add sidekiq to your gemfile, and bundle install:

```ruby
# Gemfile
# Project requirements
gem 'sidekiq'
```

```bash
bundle install
```

Now, tell padrino to mount the app, and stub some methods to make it play nice:

```ruby
# config/apps.rb
require 'sidekiq/web'

class Sidekiq::Web < ::Sinatra::Base
  class << self
    def dependencies; []; end
    def setup_application!; end
    def reload!; end
  end
end

Padrino.mount(
  'Sidekiq',
  app_class: 'Sidekiq::Web',
  app_root: Sidekiq::Web.root
).to('/sidekiq')
```

Now, create a workers directory in your app, and add all workers to
the load paths:

```ruby
# app/workers/sample_worker.rb
class SampleWorker
  include Sidekiq::Worker
  sidekiq_options retry: false

  def perform
    # Lots of hard working code....
  end
end
```

```ruby
# config/boot.rb
Padrino.before_load do
  Padrino.dependency_paths << Padrino.root('app/workers/*.rb')
  Padrino.set_load_paths('app/workers')
end
```
Add the following to your rackup config file:

```ruby
# config.ru
require 'sidekiq/web'
map('/sidekiq') { run Sidekiq::Web }
```

Add workers.rb to your config directory and tell it how to find your
workers:

```ruby
# config/workers.rb
path = File.expand_path('../../workers/*.rb', __FILE__)
Dir[path].each { |file| require file }
```

Finally, add a Procfile with the following content:

```yaml
# Procfile
web: bundle exec thin start -p $PORT
worker: bundle exec sidekiq -r ./config/workers.rb
```
