# Voter Status

> Below is an example request to obtain voter status:

```ruby
require 'net/http'
require 'addressable/uri'
require 'base64'
require 'openssl'
require 'json'

##### Example values:

# eid = '12'
# mid = 'jane@example.com'
# secret_key = 'N+vlebJgl/Lkxtu2b4hOe+JUTpVm5arWGJbQ6U7BOFs='
# (Contact support to obtain `secret_key`)

#####

def get_voter_status(secret_key, eid, mid)
  url = 'https://secure.electionbuddy.com/sso'
  headers = { 'Accept' => 'application/json' }
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

  net = Net::HTTP.new(uri.host, 443)
  net.use_ssl = true
  net.verify_mode = OpenSSL::SSL::VERIFY_PEER

  response = net.post(uri.path, uri.query, headers)
  JSON.parse(response.body)
end
```

The same endpoint can be used to obtain a voter's status. This can be used, for example, to send out your own custom vote reminders to those voters who have not yet voted.

Voter status requests use the same parameters as above, but will request a JSON response (header of: `Accept: application/json`).

### HTTP Request

`GET https://secure.electionbuddy.com/sso?{parameters}` (Header: `Accept: application/json`)

HTTP Code | Response Body | Meaning
---------- | ------- | ---------
200 | `{ 'voted' : true, 'election_state' : 'running', 'end_date' : '1590298839' }` | The only valid election state for voting is `running`: all other states mean that either voting has not begun yet, or has ended. `end_date` is the election end date as Unix Epoch (not static - may be changed by the Election Administrator).