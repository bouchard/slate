---
title: ElectionBuddy API Documentation

language_tabs: # must be one of https://git.io/vQNgJ
  - ruby
  - python
  - php

toc_footers:
  - Back to <a href='https://electionbuddy.com'>electionbuddy.com</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the SSO API documentation for ElectionBuddy (electionbuddy.com).

# Single Sign-On (SSO)

> The function/method below will generate a signed anchor `<a>` element that you can use on your organization's internal dashboard (i.e. after a member/voter logs in). You can style it as you wish to match your organization's design or theme. You'll need to pass in `eid`, `exp`, `mid`, and `signature` as appropriate.

```ruby
require 'addressable/uri'
require 'base64'
require 'openssl'

##### Example values:

# eid = '12'
# mid = 'jane@example.com'
# secret_key = 'N+vlebJgl/Lkxtu2b4hOe+JUTpVm5arWGJbQ6U7BOFs='
# (`secret_key` obtained at `https://electionbuddy.com/admin/secret-key`)

#####

def generate_vote_anchor(secret_key, eid, mid)
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
  "<a href='#{uri}' class='electionbuddy-vote-button'>Vote now</a>"
end
```

```python
import base64
import hashlib
import hmac
import time
from urllib.parse import urlparse, urlencode

##### Example values:

# eid = '12'
# mid = 'jane@example.com'
# secret_key = 'N+vlebJgl/Lkxtu2b4hOe+JUTpVm5arWGJbQ6U7BOFs='
# (`secret_key` obtained at `https://electionbuddy.com/admin/secret-key`)

#####

def generate_vote_anchor(secret_key, eid, mid):
  url = 'https://electionbuddy.com/sso'

  query_values = {
    'eid' : eid,
    'exp' : int(time.time()),
    'mid' : mid
  }

  message = urlencode(query_values)
  signature = base64.urlsafe_b64encode(
    hmac.new(secret_key.encode(), message.encode(), hashlib.sha256).digest()
  ).decode()
  query_values['signature'] = signature
  message = urlencode(query_values)
  uri = url + '?' + message
  return "<a href='" + uri + "' class='electionbuddy-vote-button'>Vote now</a>"
```

```php
<?

##### Example values:

# $eid = '12'
# $mid = 'jane@example.com'
# $secret_key = 'N+vlebJgl/Lkxtu2b4hOe+JUTpVm5arWGJbQ6U7BOFs='
# (`$secret_key` obtained at `https://electionbuddy.com/admin/secret-key`)

#####

function generate_vote_anchor($secret_key, $eid, $mid) {
  $url = 'https://electionbuddy.com/sso';

  $query_values = array(
    'eid' => $eid,
    'exp' => time(),
    'mid' => $mid
  );

  $message = http_build_query($query_values);
  $signature = strtr(base64_encode(hash_hmac('sha256', $message, $secret_key, true)), '+/', '-_');
  $query_values['signature'] = $signature;
  $message = http_build_query($query_values);
  $uri = $url . '?' . $message;

  return ("<a href='" . $uri . "' class='electionbuddy-vote-button'>Vote now</a>");

}
?>
```

This endpoint authenticates a voting request for a particular user (for example, from your organization's internal dashboard). If the signature is valid and the voter (identified by `mid`) has yet to vote, they will be forwarded to a fresh ballot for the election `eid`.

### HTTP Request

`GET https://electionbuddy.com/sso?{parameters}`

### Query Parameters

Parameter | Description
--------- | -----------
eid | Election ID: when editing your election, your election ID is the numeral that appears in the URL. (`1234` in `https://electionbuddy.com/elections/1234/edit`)
exp | Request expiration date, in [Unix Epoch Time](https://www.epochconverter.com/). This time must be +/- 5 minutes from the time the request is received. If `exp` is older than 5 minutes, the voter will be directed to a "Link Expired" page.
mid | Member ID: A unique identifier for the member who is voting, such as an email address, or a membership ID, or a database primary key. This string can be anything that you can guarantee is unique for each of your voters. If a ballot attached to this member ID is already on your ElectionBuddy election voter list, this ballot will be shown to the authenticated voter. If there is no voter on your ElectionBuddy election voter list with this member ID, a fresh ballot will be created with an anonymized ballot ID.
signature | Generated signature using `secret_key`. See below.

* All requests must be signed by `signature` **appended** (as the last parameter) to the query string, and generated from the formatted query string consisting of `eid`, `exp`, and `mid`.
* Signatures must be generated using HMAC-SHA256 using the `secret_key` for your election in your [ElectionBuddy account](https://electionbuddy.com/accounts/secret). Secret keys are unique to your `Election`.
* All other parameters should appear **alphabetically** in the query string (i.e. `eid`, `exp`, `mid`).

<aside class="notice">
Remember - the query parameters must occur <strong>in order (alphabetically)</strong> as shown above, with the signature <strong>appended</strong> as the last parameter of the query string.
</aside>
