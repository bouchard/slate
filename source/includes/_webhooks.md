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

The HTTP POST requests will contain a JSON payload. The payload contains
information about your vote, the organization associated with that vote,
and the current status of the vote. The parts of the payload will be
different depending on your vote configuration.

Voter information, such as their postal address, SMS number and email address
will be included if that information is included in the voter list.

The `opened_at`, `completed_at`, etc. fields represent instants in time when the
given event happened. If the gien event has happened then the value with be a
UNIX timestamp. That is, a number of *seconds* after January 1st, 1970.
If the given event has not happened, the value will be `null`.

## Voter Choices Overview

```json
{
  // ... other fields ...
  "questions": [
    {
      "question": "Bylaw Amendment Approval of Article XLII",
      "type": "plurality",
      "answers": {
        "regular": [
          {
            "title": "Yes - I approve the amendments",
            "choice": "true"
          }
        ],
        "write_ins": []
      }
    }
  ]
}
```

If your Voter Anonymity settings allow administrators to view voter choices,
then voter choices will be included in the payload.

![radio buttons offering three choices: Secret Ballot, Poll, and Show of Hands](voter-choices.png)

There are six types of questions corresponding to the 6 voting system options
available when adding Positions/Questions.

![A drop down offering 6 choices: Plurality, Cumulative, Preferential, Approval, Nomination, and Scored](voting-system-options.png)

Correspondingly, The `"type"` field on each object in the `"questions"` array
will have one of the following values:

* `"plurality"`
* `"cumulative"`
* `"preferential"`
* `"approval"`
* `"nomination"`
* `"scored"`

The `"answers"` field will have slightly different contents depending on the
`"type"`. The overal shape will be the same but the `"choice"` fields,
on the objects inside the `"regular"` and/or `"write_ins"` arrays, will be have
different possible values and the proper way to interpret those values will be
different.

`"title"` will always contain the name of the option the voter selected. For
example, the name of the candidate.

If write-in answers are allowed, (not available for all question types), then
any write-in answers will appear in the `"write_ins"`array. All other answers
will show up in the `"regular"`array. The written in answer will appear in the
`"title"` field.

If multiple options were selected then multiple objects can show up
in the appropriate array.

## Plurality

```json
{
  // ... other fields ...
  "questions": [
    {
      "question": "Council Leader",
      "type": "plurality",
      "answers": {
        "regular": [
          {
            "title": "Jane Doe",
            "choice": "true"
          },
          {
            "title": "John Doe",
            "choice": "true"
          }
        ],
        "write_ins": [
          {
            "title": "A. N. Other",
            "choice": "true"
          }
        ]
      }
    }
  ]
}
```

For this question type, `"choice"` will always be `"true"`, assuming abtaining
is not allowed. Otherwise, see [the Abstention section](#webhooks-abstention).

## Cumulative

```json
{
  // ... other fields ...
  "questions": [
    {
      "question": "Council Leader",
      "type": "cumulative",
      "answers": {
        "regular": [
          {
            "title": "Jane Doe",
            "choice": "2"
          },
          {
            "title": "John Doe",
            "choice": "1"
          }
        ],
        "write_ins": [
          {
            "title": "A. N. Other",
            "choice": "1"
          }
        ]
      }
    }
  ]
}
```

For this question type, `"choice"` will be a number, (represented as a string),
assuming abtaining is not allowed. Otherwise, see
[the Abstention section](#webhooks-abstention).

The numbers here represents how many votes the voter assigned to the option.

Options with zero votes assigned will not show up.

## Preferential

```json
{
  // ... other fields ...
  "questions": [
    {
      "question": "Council Leader",
      "type": "preferential",
      "answers": {
        "regular": [
          {
            "title": "Jane Doe",
            "choice": "2"
          },
          {
            "title": "John Doe",
            "choice": "1"
          }
        ],
        "write_ins": [
          {
            "title": "A. N. Other",
            "choice": "3"
          }
        ]
      }
    }
  ]
}
```

For this question type, `"choice"` will be a number, (represented as a string),
assuming abtaining is not allowed. Otherwise, see
[the Abstention section](#webhooks-abstention).

The numbers here represents the relavtive ordering of options that was selected
by the voter. Note that, *unlike* the numbering on the ballot, larger numbers
indicate *more preferred* options, and lower numbers indicate *less preferred*
options.

## Approval

```json
{
  // ... other fields ...
  "questions": [
    {
      "question": "Council Leader",
      "type": "approval",
      "answers": {
        "regular": [
          {
            "title": "Jane Doe",
            "choice": "true"
          },
          {
            "title": "John Doe",
            "choice": "true"
          }
        ],
        "write_ins": []
      }
    }
  ]
}
```

For this question type, `"choice"` will always be `"true"`, assuming abtaining
is not allowed. Otherwise, see [the Abstention section](#webhooks-abstention).

Note that write-ins are not permitted for Approval voting, but we still send
down a `"write_ins"` array.

## Nomination

```json
{
  // ... other fields ...
  "questions": [
    {
      "question": "Council Leader",
      "type": "nomination",
      "answers": {
        "regular": [],
        "write_ins": [
          {
            "title": "Jane Doe",
            "choice": "true"
          },
          {
            "title": "A. N. Other",
            "choice": "true"
          }
        ]
      }
    }
  ]
}
```

For this question type, `"choice"` will always be `"true"`, assuming abtaining
is not allowed. Otherwise, see [the Abstention section](#webhooks-abstention).

Note that regular votes do not occur for Nomination voting, except for when a
voter [abstains](#webhooks-abstention), (if enabled) but we sill send down
a `"regular"` array.

## Scored

```json
{
  // ... other fields ...
  "questions": [
    {
      "question": "Member Feedback: $4300 Excess Budget Spending",
      "type": "scored",
      "answers": {
        "regular": [
          {
            "title": "Increase security/surveillance hours",
            "choice": "3"
          },
          {
            "title": "Hire a caterer for the Annual General Meeting",
            "choice": "5"
          }
        ],
        "write_ins": []
      }
    }
  ]
}
```

For this question type, `"choice"` will either be a number, (represented as a
string), or the string `"not_applicable"`, assuming abtaining is not allowed.
Otherwise, see [the Abstention section](#webhooks-abstention). The string
`"not_applicable"` only shows up if the question allows that as an option.

The numbers here directly correspond to the numbers the voter selected on the
ballot. Multiple scores will appear within the same object in the `"questions"`
array, if multiple scores are asked for on the same question.

The default scale, without customization, is as follows:

* 1: Strongly disagree
* 2: Disagree
* 3: Neither agree nor disagree
* 4: Agree
* 5: Strongly agree

Note that write-ins are not permitted for Scored voting, but we still send
down a `"write_ins"` array.

## Absention

```json
{
  // ... other fields ...
  "questions": [
    {
      "question": "Executive Board",
      "type": "nomination",
      "answers": {
        "regular": [
          {
            "title": "abstain",
            "choice": "abstain"
          }
        ],
        "write_ins": []
      }
    }
  ],
}
```

If a question allows abstaining, then no matter what type of question it is,
when a voter abstains, then an object with a `"choice"` field with the value
`"abstain"` will be placed in the `"regular"` array.
