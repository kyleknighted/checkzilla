## CheckZilla

CheckZilla is a command line tool allowing you to check and be notified of outdated software.
CheckZilla is extensible and already supports multiple "Checkers" (RubyGem, Pacman, Npm) and "Notifiers" (Console, Email, HipChat, notify-send).

The main usage currently is to use it as a CRON notifying you everyday of new softwares updates.

**Warning** This is Beta Software !

## How to use it

```
$> checkzilla your-config-file
```

The configuration is using a DSL, here's a sample :

```ruby
CheckZilla::Model.new('This is the title of my report') do

  check_updates :rubygem do |rubygem|
    rubygem.path = "/home/mike/code/diaspora"
  end

  notify_by :hipchat do |hipchat|
    hipchat.api_token = '95def4314870443f46b8c4694bd88e'
    hipchat.room = 'test'
    hipchat.username = 'CheckUpdates'
  end

  notify_by :console
end
```

## Checkers

Checkers are defined via a `CheckZilla::Check::NEW_CHECKER` class, they need to define 2 methods:

`initialize(&block)` returns self

`perform!` fills `@results` with `@results[software_name] = [software_current_version, software_newer_version]`

Here's the list of availables checkers:

### Rubygem

```ruby
check_updates :rubygem do |rubygem|
  rubygem.path = "/home/mike/code/diaspora"
end
```

Tries to find a `Gemfile.lock` if `path` is defined, otherwise will use `gem list` for a system wide ruby installation. It matches your dependencies against the rubygems api to find what's outdated.
  
### Pacman

```ruby
check_updates :pacman
```

Tries to determine your outdated package via:

`sudo pacman -Sy > /dev/null ; package-query -AQu -f '%n %l %V'`

**Warning** You need to execute checkzilla as root as I didn't find a better way to ask pacman to synchronise the db. `package-query` is required (it's a dependency of yaourt).

### Npm

```ruby
check_updates :npm do |npm|
  npm.path = "/home/mike/code/nodes3"
end
```

It requires `path` and will determine the dependencies via `npm outdated`

## Notifiers

Notifiers are defined via a `CheckZilla::Notifier::NEW_NOTIFIER` class, they need to define 2 methods:

`initialize(&block)` returns self

`perform!(checkers)` loop other the checkers, format the `results` and notifies you.

Here's the list of availables notifiers:

### Console

```ruby
notify_by :console
```

Outputs to the console.

### Twitter

```ruby
notify_by :twitter do |twitter|
  twitter.consumer_key = YOUR_CONSUMER_KEY
  twitter.consumer_secret = YOUR_CONSUMER_SECRET
  twitter.oauth_token = YOUR_OAUTH_TOKEN
  twitter.oauth_token_secret = YOUR_OAUTH_TOKEN_SECRET
end
```

### Email

```ruby
notify_by :email do |email|
  email.pony_settings = {
    :to => 'you@gmail.com',
    :subject => 'BOT Report',
    :from => 'you+bot@gmail.com',
    :via => :smtp,
    :via_options => {
      :address              => 'smtp.gmail.com',
      :port                 => '587',
      :enable_starttls_auto => true,
      :user_name            => 'you@gmail.com',
      :password             => 'yourpassword',
      :authentication       => :plain,
      :domain               => "localhost.localdomain"
    }
  }
end
```

Send you an email using [Pony](https://github.com/benprew/pony)

### Hipchat

```ruby
notify_by :hipchat do |hipchat|
  hipchat.api_token = '95def4314870443f46b8c4694bd88e'
  hipchat.room = 'test'
  hipchat.username = 'CheckUpdates'
end
```

Sends you a notification to the [HipChat](http://hipchat.com) room of your choice

### notify-send

```ruby
notify_by :notify_send
```

Send a desktop notification (only tested on archlinux/xfce but should work on ubuntu/unity).

## TODO

- Persistence layer (no need to be fancy, a .yml could do it) so you can send incremental updates
- Add notifiers global options like `only` and `except`. They'd both take an array of checkers.
- Add notifier global option `template` ? So you can pimp your emails and Twitter mentions
- Notifiers: Growl, kdialog, basecamp, jabber
- Checkers: Homebrew, apt-get
- Rack Application and/or HTML report

## Helping out

* Feedback is good
* Issue reporting is better
* Pull requests are way better
* Awesome pull requests are *obviously awesome*