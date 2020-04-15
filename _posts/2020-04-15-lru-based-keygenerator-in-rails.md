---
layout: post
title: LRU-Cache based KeyGenerator in Rails
description: 
summary: 
comments: true
tags: [rails, cache, cryptography]
---

Active Support component of Ruby on Rails provides a class named [`KeyGenerator`](https://api.rubyonrails.org/classes/ActiveSupport/KeyGenerator.html) for generating secret keys. This is a wrapper around OpenSSL's implementation of [PBKDF2](https://en.wikipedia.org/wiki/PBKDF2) (Password-Based Key Definition Function 2) and is commonly used to generate secrets keys for encryption use-cases. 

[`CachingKeyGenerator`](https://api.rubyonrails.org/classes/ActiveSupport/CachingKeyGenerator.html) is a wrapper around [`KeyGenerator`](https://api.rubyonrails.org/classes/ActiveSupport/KeyGenerator.html) class which helps us to avoid re-executing the key generation process when it is called using same salt and the key_size. This is really helpful because key generation process is computationally costly.  However, [`CachingKeyGenerator`](https://api.rubyonrails.org/classes/ActiveSupport/CachingKeyGenerator.html) uses an instance of `Concurrent::Map` to store the generated keys in memory to avoid the re-execution. This is not really helpful when the number of keys to be cached is large so that periodic cleanup is required on least recently used ones. 

Assume that we are using [`CachingKeyGenerator`](https://api.rubyonrails.org/classes/ActiveSupport/CachingKeyGenerator.html) to generate unique secret keys for encrypting personal information of our application's users. This will return the key if it exists in the `Concurrent::Map` or re-execute the key generation process and cache the key before returning to caller. When the userbase is huge, it is not practical to store all the generated keys in the memory of application instances. Instead it would be ideal to perform a periodic clean up of the cached keys to avoid memory hogging. 

We will see how we can solve this issue using a new wrapper around [`KeyGenerator`](https://api.rubyonrails.org/classes/ActiveSupport/KeyGenerator.html). Instead of `Concurrent::Map` in [`CachingKeyGenerator`](https://api.rubyonrails.org/classes/ActiveSupport/CachingKeyGenerator.html), this implementation will use [`ActiveSupport::Cache::MemoryStore`](https://api.rubyonrails.org/classes/ActiveSupport/Cache/MemoryStore.html) which, as per the documentation, is Thread-safe and has an LRU based clean-up mechanism built in:

> This cache has a bounded size specified by the `:size` options to the initializer (default is 32Mb). When the cache exceeds the allotted size, a cleanup will occur which tries to prune the cache down to three quarters of the maximum size by removing the least recently used entries.

```ruby
class LruCachingKeyGenerator
  KEY_EXPIRY = 1.day  # expire the keys which are older than a day

  def initialize(key_generator)
    @key_generator = key_generator
    @keys_cache = ActiveSupport::Cache::MemoryStore.new(expires_in: KEY_EXPIRY)
  end

  # Returns a derived key suitable for use.
  def generate_key(*args)
    cached_key = @keys_cache.fetch(args.join)
    return cached_key unless cached_key.nil?

    # cache miss, generate a new key
    generated_key = @key_generator.generate_key(*args)
    @keys_cache.write(args.join, generated_key)
    return generated_key
  end
end
```

As the default cache size is 32Mb, this helps avoiding memory hogging of our application while providing a cache for frequently used keys.