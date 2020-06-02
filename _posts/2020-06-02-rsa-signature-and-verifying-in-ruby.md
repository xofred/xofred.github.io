---
title:  "RSA Signature and Verifying in Ruby"
tags: crytography rsa ruby

---

*RSA (Rivest–Shamir–Adleman) is one of the first public-key cryptosystems and is widely used for secure data transmission*

Generate a key pair, the server will use the public key, the client will use the private key.

1. Client will generate a signature using the private key![{\displaystyle h={\text{hash))(m);}](https://wikimedia.org/api/rest_v1/media/math/render/svg/0be40a515a0de48b77142793f0fd0ae186bfb924)
2. Server will verify the signature using the public key![{\displaystyle (h^{e})^{d}=h^{ed}=h^{de}=(h^{d})^{e}\equiv h{\pmod {n))}](https://wikimedia.org/api/rest_v1/media/math/render/svg/11546205f3d9d859bf7e917ee746268ac50b6729)

```ruby
require 'openssl'

# Generate the key pairs, and give client the private key
pkey = OpenSSL::PKey::RSA.new(2048)
private_key = pkey.to_pem # or to_der, depends on client

# Client
data = "Sign me!"
pkey = OpenSSL::PKey::RSA.new(private_key)
signature = pkey.sign("SHA256", data)

# Server
pub_key = pkey.public_key
pub_key.verify("SHA256", signature, data) # true
```

[RSA (cryptosystem)](https://www.wikiwand.com/en/RSA_(cryptosystem))

[OpenSSL::PKey::RSA](http://ruby-doc.org/stdlib-2.7.1/libdoc/openssl/rdoc/OpenSSL/PKey/RSA.html)
