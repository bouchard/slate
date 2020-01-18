# Voter Status

> Below is an example request to obtain voter status:

```ruby
require 'addressable/uri'
require 'base64'
require 'openssl'

##### Example values:

# eid = '12'
# mid = 'jane@example.com'
# secret_key = 'N+vlebJgl/Lkxtu2b4hOe+JUTpVm5arWGJbQ6U7BOFs='
# (Contact support to obtain `secret_key`)

#####

def get_voter_status(secret_key, eid, mid)
  url = 'https://electionbuddy.com/sso'

  query_values = {
    eid: eid,
    exp: Time.now.to_i,
    mid: mid
  }

  uri = Addressable::URI.parse(url)
  uri.query_values = query_values
  message = uri.query
  signature = Base64.urlsafe_encode64(
    OpenSSL::HMAC.digest('sha256', secret_key, message)
  )
  uri.query_values = query_values.merge(signature: signature)
  net = Net::HTTP.new(uri)
  net.use_ssl = true
  net.headers = { 'Accept' => 'application/json' }
  res = net.get_response
  JSON.parse(res.body)
end
```

The same endpoint can be used to obtain a voter's status (`{ 'voted' : true|false }`). This can be used, for example, to send out your own custom vote reminders to those voters who have not yet voted.

Voter status requests use the same parameters as above, but will request a JSON response (header of: `Accept: application/json`).

### HTTP Request

`GET https://electionbuddy.com/sso?{parameters}` (Header: `Accept: application/json`)

### HTTP Response

* HTTP code: 200
* Response body: `{ 'voted' : true|false }`
* Error codes same as above.
