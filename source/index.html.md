---
title: Voting Integrations (VI) Documentation

language_tabs: # must be one of https://git.io/vQNgJ
  - ruby
  - python
  - php

toc_footers:
  - Back to <a href='https://electionbuddy.com'>electionbuddy.com</a>

includes:
  - errors
  - voter-status

search: true
---

# Introduction

Welcome to the Voting Integrations (VI) documentation for ElectionBuddy (electionbuddy.com). We continue to maintain both our legacy (version 1, or V1) integrations, as well our current (version 2, or V2) voting integrations to allow your voters to authenticate with ElectionBuddy within your own member or customer portal or customer relationship management (CRM) software.

**V1** focused on providing you with a method to generate a signed URL to send your voters to a specific election. **V2** expands on this by allowing you to give your voters a list of all running elections in which they are eligible to vote, and allowing them to vote in any or all of them in turn while being authenticated behind your customer portal.

# Current (Version 2)

> The function/method below will return templated HTML in the form of a `<ul>` list of elections that the current voter is eligible to vote in. It will search all `Elections` belonging to an `Organization`, **including** sister `Organization`s (i.e. sharing the same `BillingAccount`).

```ruby
require 'addressable/uri'
require 'base64'
require 'openssl'
require 'faraday'

#####

# Parameters:
# `oid` - your Organization's ID (available from https://secure.electionbuddy.com/organizations)
# `exp` - Unix epoch timestamp (we validate this to within +/- 5 minutes of current time)
# `identifier` - the Voter's identifier for which to search for eligible elections
# `secret_token` - your Organizations Secret Token (https://secure.electionbuddy.com/organizations)
# `signature` - a Base64, HMAC-SHA256 signature of the above parameters in order (exp, identifier, oid)
#
# Returns HTML:
#

#####

def get_election_list(oid, exp, identifier, secret_token)
  url = 'https://secure.electionbuddy.com/integrations/v2/elections'

  query_values = {
    exp: Time.now.to_i,
    identifier: identifier,
    oid: oid
  }

  uri = Addressable::URI.parse(url)
  uri.query_values = query_values
  message = uri.query
  signature = Base64.urlsafe_encode64(
    OpenSSL::HMAC.digest('sha256', secret_token, message)
  )
  uri.query_values = query_values.merge(signature: signature)
  Faraday.get(uri).body
end
```

```python
import base64
import hashlib
import hmac
import time
import requests
from urllib.parse import urlparse, urlencode

#####

# Parameters:
# `oid` - your Organization's ID (available from https://secure.electionbuddy.com/organizations)
# `exp` - Unix epoch timestamp (we validate this to within +/- 5 minutes of current time)
# `identifier` - the Voter's identifier for which to search for eligible elections
# `secret_token` - your Organizations Secret Token (https://secure.electionbuddy.com/organizations)
# `signature` - a Base64, HMAC-SHA256 signature of the above parameters in order (exp, identifier, oid)
#
# Returns HTML:
#

#####

def get_election_list(oid, exp, identifier, secret_token):
  url = 'https://secure.electionbuddy.com/integrations/v2/elections'

  query_values = {
    'exp' : int(time.time()),
    'identifier' : identifier,
    'oid' : oid
  }

  message = urlencode(query_values)
  signature = base64.urlsafe_b64encode(
    hmac.new(secret_token.encode(), message.encode(), hashlib.sha256).digest()
  ).decode()
  query_values['signature'] = signature
  message = urlencode(query_values)
  uri = url + '?' + message
  return requests.get(uri).content
```

```php
<?

#####

# Parameters:
# `oid` - your Organization's ID (available from https://secure.electionbuddy.com/organizations)
# `exp` - Unix epoch timestamp (we validate this to within +/- 5 minutes of current time)
# `identifier` - the Voter's identifier for which to search for eligible elections
# `secret_token` - your Organizations Secret Token (https://secure.electionbuddy.com/organizations)
# `signature` - a Base64, HMAC-SHA256 signature of the above parameters in order (exp, identifier, oid)
#
# Returns HTML:
#

#####

function get_election_list($oid, $exp, $identifier, $secret_token) {
  $url = 'https://secure.electionbuddy.com/integrations/v2/elections';

  $query_values = array(
    'exp' => time(),
    'identifier' => $identifier,
    'oid' => $oid
  );

  $message = http_build_query($query_values);
  $signature = strtr(base64_encode(hash_hmac('sha256', $message, $secret_token, true)), '+/', '-_');
  $query_values['signature'] = $signature;
  $message = http_build_query($query_values);
  $uri = $url . '?' . $message;

  return file_get_contents($uri);

}
?>
```

This endpoint authenticates a voting request for a particular organization. If the signature is valid and the voter (identified by `identifier`) has yet to vote in any elections they are eligible to vote in, we will return an HTML list (`<ul>`) with links to vote.

***It is very important that the voter identifiers you use (for example, membership number) are consistent across any elections your organization(s) may run. The list this endpoint presents to voters is based on a search by identifier &mdash; if, for example, identifier `123456` represents Voter A in one election, and Voter B in another, that will allow either Voter A or B to vote for each other in both elections. You almost certainly do not want this! Please ensure your voter identifiers are globally unique.***

### HTTP Request

`GET https://secure.electionbuddy.com/integrations/v2/elections?{parameters}`

### Query Parameters

Parameter | Description
--------- | -----------
exp | Request expiration date, in [Unix Epoch Time](https://www.epochconverter.com/). This time must be +/- 30 seconds from the time the request is received.
identifier | Member ID: A unique identifier for the member who is voting, such as an email address, or the `identifier` when you uploaded your voter details for an election.
oid | Your Organization's ID (available from https://secure.electionbuddy.com/organizations)
signature | Generated signature using `secret_token`. See below.

* All requests must be signed by `signature` **appended** (as the last parameter) to the query string, and generated from the formatted query string consisting of `oid`, `exp`, and `identifier`.
* Signatures must be generated using HMAC-SHA256 using the `secret_token` for your Organization in your [ElectionBuddy account](https://secure.electionbuddy.com/organizations).
* All other parameters should appear **alphabetically** in the query string (i.e. `exp`, `identifier`, `oid`).

<aside class="notice">
Remember - the query parameters must occur <strong>in order (alphabetically)</strong> as shown above, with the signature <strong>appended</strong> as the last parameter of the query string.
</aside>


# Legacy (Version 1)

> The function/method below will generate a signed anchor `<a>` element that you can use on your organization's internal dashboard (i.e. after a member/voter logs in). You can style it as you wish to match your organization's design or theme. You'll need to pass in `eid`, `exp`, `mid`, and `signature` as appropriate.

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

def generate_vote_anchor(secret_key, eid, mid)
  url = 'https://secure.electionbuddy.com/integrations/v1/sso'

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
# (Contact support to obtain `secret_key`)

#####

def generate_vote_anchor(secret_key, eid, mid):
  url = 'https://secure.electionbuddy.com/integrations/v1/sso'

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
# (Contact support to obtain `secret_key`)

#####

function generate_vote_anchor($secret_key, $eid, $mid) {
  $url = 'https://secure.electionbuddy.com/integrations/v1/sso';

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

`GET https://secure.electionbuddy.com/integrations/v1/sso?{parameters}`

### Query Parameters

Parameter | Description
--------- | -----------
eid | Election ID: when editing your election, your election ID is the numeral that appears in the URL. (`1234` in `https://secure.electionbuddy.com/elections/1234/edit`)
exp | Request expiration date, in [Unix Epoch Time](https://www.epochconverter.com/). This time must be +/- 5 minutes from the time the request is received. If `exp` is older than 5 minutes, the voter will be directed to a "Link Expired" page.
mid | Member ID: A unique identifier for the member who is voting, such as an email address, or a membership ID, or a database primary key. This string can be anything that you can guarantee is unique for each of your voters. If a ballot attached to this member ID is already on your ElectionBuddy election voter list, this ballot will be shown to the authenticated voter. If there is no voter on your ElectionBuddy election voter list with this member ID, a fresh ballot will be created with an anonymized ballot ID.
signature | Generated signature using `secret_key`. See below.

* All requests must be signed by `signature` **appended** (as the last parameter) to the query string, and generated from the formatted query string consisting of `eid`, `exp`, and `mid`.
* Signatures must be generated using HMAC-SHA256 using the `secret_key` for your election in your [ElectionBuddy account](https://secure.electionbuddy.com/accounts/secret). Secret keys are unique to your `Election`.
* All other parameters should appear **alphabetically** in the query string (i.e. `eid`, `exp`, `mid`).

<aside class="notice">
Remember - the query parameters must occur <strong>in order (alphabetically)</strong> as shown above, with the signature <strong>appended</strong> as the last parameter of the query string.
</aside>
