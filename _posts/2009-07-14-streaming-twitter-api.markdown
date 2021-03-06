---
layout: post
date: 2009-07-14
comments: false
sharing: false
---
<p>
  I have been researching the <a href="http://apiwiki.twitter.com/Streaming-API-Documentation" target="_blank" rel="nofollow">Twitter Streaming API</a> recently for a side project (secret) and was playing around with it in Ruby to get a feel for how it works. I started out using an <a href="http://github.com/igrigorik/em-http-request/tree/master" target="_blank" rel="nofollow">asynchronous http client</a> to take advantage of the API but wound up having some issues with how it was chunking json as a response type (.xml was fine).
</p>
<p>
  I wound up stumbling upon Brian Lopez's <a href="http://github.com/brianmario/yajl-ruby/tree/master" target="_blank" rel="nofollow">yajl-ruby</a> which is a ruby binding for a native streaming json library. It has excellent support for dealing with streams and also supports persistent http connections (perfect for the streaming API).
</p>
<p>
  Below is some code utilizing yajl that opens a connection and pushes the screen name and text of a tweet to the console as it is received. (disclaimer: This is not even close to being production-ready as Twitter has strong recommendations for reconnect logic and dealing with the API's alpha QOS). This is using the publicly available sprinkler stream; which is only a subsection of the public timeline but should be good enough for general application use (there are gardenhose and firehose streams available for statistical purposes but access to these is at Twitter's discretion).
</p>

```ruby

require "uri"
require "yajl/http_stream"

u = "twitter username here"
p = "twitter password here"

url = URI.parse("http://#{u}:#{p}@stream.twitter.com/spritzer.json")
Yajl::HttpStream.get(url, :symbolize_keys => true) do |status|
  unless status.has_key?(:delete)
    puts "screen_name: #{status[:user][:screen_name]}"
    puts "message: #{status[:text]}"
    puts
  end
end
```
<p>
  And that is it. I can point out that the streams will additionally push delete operations out that are to be honored by client applications. Above I am filtering out the delete messages by key name and only printing out actual statuses.
</p>
