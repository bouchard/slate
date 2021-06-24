# Webhooks

Our webhooks feature allows a URL, supplied by you, to receive HTTP POST
requests corresponding to events related to your vote(s). For example, when
individual votes are recorded.

The URL is configured through the Electionbuddy website, if enabled on your
account. To enable webhooks contact an Electionbuddy customer support
representative.

Once enabled, individual webhooks can be enabled for all elections, or enabled or disabled
on an election-by-election basis, in the organization settings.

![a UI for configuring overall webhook settings](webhook-organization-settings.png)

If enabled on an election-by-election basis, the webhooks can be configured on
the voter list section fo the vote setup.

![a UI for configuring webhook settings for a single vote](webhook-vote-settings.png)

## Payload Overview

The HTTP POST requests will contain a JSON payload. The payload contains
information about your vote, the organization associated with that vote,
and the current status of the vote. The parts of the payload will be
different depending on your vote configuration.


```json
{
    "organization_id": 1234,
    "organization_name": "My organization",
    "organization_owner_name": "Jane Doe",
    "organization_owner_email": "support@electionbuddy.com",
    "election_id": 12345,
    "election_name": "My election",
    "identifier": "1",
    "email": "voter@example.com",
    "sms": "555-5555",
    "postal": "1, 3984 Massachusetts Avenue, Washington DC",
    "status": [
        "Voted",
        "Key Surfaced"
    ],
    "opened_at": 1600000000,
    "completed_at": 1600001234,
    "added_at": null,
    "spoiled_at": null,
    "spoil_reason": null,
    "key_surfaced_at": null,
    "ballot_printed_at": null,
    "ip": "0.0.0.0",
    "unsubscribed_at": null
    // Non-exhaustive.
}
```

Voter information, such as their postal address, SMS number and email address
will be included if that information is included in the voter list.

The `opened_at`, `completed_at`, etc. fields represent instants in time when the
given event happened. If the gien event has happened then the value with be a
UNIX timestamp. That is, a number of *seconds* after January 1st, 1970.
If the given event has not happened, the value will be `null`.

## Voter Choices

If your Voter Anonymity settings allow administrators to view voter choices,
then voter choices will be included in the payload.

![radio buttons offering three choices: Secret Ballot, Poll, and Show of Hands](voter-choices.png)

```json
{
  // ... other fields ...
  "questions": [
    {
      "question_id": "123",
      "question_text": "Bylaw Amendment Approval of Article XLII",
      "answers": [
        {
          "answer_id": "1234",
          "answer_text": "Yes - I approve the amendments",
        }
      ]
    }
  ],
}
```
