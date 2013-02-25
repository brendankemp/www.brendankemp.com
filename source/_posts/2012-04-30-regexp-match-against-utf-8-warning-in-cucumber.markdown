---
layout: post
title: "Warning: Regexp match against UTF-8 in Cucumber, Rails 3.1.4"
date: 2012-04-30 00:51
comments: true
categories: [rails, ruby]
---

After upgrading to Rails 3.1.4 and Ruby 1.9.3, running features produced dozens of instances of this warning:

    activesupport-3.1.4/lib/active_support/core_ext/string/output_safety.rb:23: warning: regexp match /.../n against to UTF-8 string


That's not very fun to look at, so I cracked open activerecord:

``` ruby
# activesupport-3.1.4/lib/active_support/core_ext/string/output_safety.rb
class ERB
  module Util
    # ...
    def html_escape(s)
      s = s.to_s
      if s.html_safe?
        s
      else
        s.gsub(/[&"><]/n) { |special| HTML_ESCAPE[special] }.html_safe
      end
    end
    # ...
  end
end
```

You can see the offending line gsubbing out characters that should be escaped.
Fortunately, there already is [a commit](https://github.com/rails/rails/commit/63cd9432265a32d222353b535d60333c2a6a5125)
for this issue, and it has been backported to the 3-1-stable branch, it just hasn't been released yet.
Until it is released, if we don't want to look at those UTF-8 warnings, we can use the fix for a monkey patch
(1.9 only, use the other half of the aforelinked commit if you are on 1.8):

``` ruby
# lib/core_ext/erb/html_escape.rb
class ERB
  module Util
    def html_escape(s)
      s = s.to_s
      if s.html_safe?
        s
      else
        s.encode(s.encoding, :xml => :attr)[1...-1].html_safe
      end
    end
    alias h html_escape
    module_function :h
    module_function :html_escape
  end
end
```

The commit that fixes this in activesupport should be released with the next version of Rails, so we can give ourselves
reminder to remove this patch once we update:

``` ruby
# spec/controllers/application_controller_spec.rb

describe ApplicationController do
  it "should not include the html_escape.rb patch after upgrading to Rails 3.1.5"
end
```

Props to [this guy](http://stackoverflow.com/a/7189698/86630) on StackOverflow for the spec reminder idea.
