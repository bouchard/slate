# Errors

The ElectionBuddy APIs return the following error codes:


Error code | Meaning
---------- | -------
422 | Unprocessable Entity -- Your request is valid but we were unable to create a ballot based on your request. This usually happens when your election has reached its maximum number of voters.
404 | Not Found -- The election does not exist, the voter is not found, or the voter is not authorized.
