---
layout: post
title: "padrino with sidekiq"
date: 2014-07-30 14:30:00
---

i recently added [sidekiq](https://github.com/mperham/sidekiq) to a
[padrino](https://github.com/padrino/padrino-framework) app. mounting
the sinatra app wasn't completely straightforward, and i didn't find
everything i needed in one place. so here's an attempt to do just
that.

first, add sidekiq to your gemfile, and bundle install:

```ruby
# Gemfile
# Project requirements
gem 'sidekiq'
```

```bash
bundle install
```

now, tell padrino to mount the app, and stub some methods to make it play nice:

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
  :app_class => 'Sidekiq::Web',
  :app_root => Sidekiq::Web.root
).to('/sidekiq')
```

now, create a workers directory in your app, and add all workers to
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
add the following to your rackup config file:

```ruby
# config.ru
require 'sidekiq/web'
map('/sidekiq') { run Sidekiq::Web }
```

add workers.rb to your config directory and tell it how to find your
workers:

```ruby
# config/workers.rb
path = File.expand_path('../../workers/*.rb', __FILE__)
Dir[path].each { |file| require file }
```

finally, add a Procfile with the following content:

```yaml
# Procfile
web: bundle exec thin start -p $PORT
worker: bundle exec sidekiq -r ./config/workers.rb
```
