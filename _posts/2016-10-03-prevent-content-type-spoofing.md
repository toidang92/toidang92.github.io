---
layout: post
title: Prevent Content Type Spoofing on Paperclip
---

## Understanding the problem

If you have ever used `paperclip`, maybe you have seen the message like that: **Image has contents that are not what they are reported to be**.

For this bug, we will have two solutions:

* Specify an extension that cannot otherwise be mapped:

{% highlight ruby %}
Paperclip.options[:content_type_mappings] = {
  pem: "text/plain"
}
{% endhighlight %}

* Override `media_type_spoof_detector` method:

{% highlight ruby %}
require 'paperclip/media_type_spoof_detector'
module Paperclip
  class MediaTypeSpoofDetector
    def spoofed?
      false
    end
  end
end
{% endhighlight %}

The second solution is very bad. Why? The monkey-patching will ignore check type of uploading file, if an attacker put an entire HTML page into the EXIF tag of a completely valid JPEG and named the file “gotcha.html,” they could potentially trick users into an XSS vulnerability.

## How do paperclip determine Content Type Spoofing

{% highlight ruby %}
def spoofed?
  if has_name? && has_extension? && media_type_mismatch? && mapping_override_mismatch?
    Paperclip.log("Content Type Spoof: Filename #{File.basename(@name)} (#{supplied_content_type} from Headers, #{content_types_from_name.map(&:to_s)} from Extension), content type discovered from file command: #{calculated_content_type}. See documentation to allow this combination.")
    true
  else
    false
  end
end
{% endhighlight %}

[FULL CODE](https://github.com/thoughtbot/paperclip/blob/e60f00027704298455c039e111d96bcf46e12822/lib/paperclip/media_type_spoof_detector.rb#L13-L20)

1. has\_name?: check file name exists
2. has\_extension?: check file extension exists
3. media\_type\_mismatch?: did content type include defined list by file name?
4. mapping\_override\_mismatch?: content type discovered from file command

```shell
file -b --mime '/var/folders/w1/jljslm493yd9gfvbddn4ghp80000gn/T/224e20c3e580e19e06486bc62811c72d20161002-13093-q2f599.png'
```

**Note**

**You should use code from verified source. Otherwise, you should try to understand what the code will do.**

