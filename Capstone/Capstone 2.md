
Service running on port 3000

To reset /controls/reset
/controls/sampleData --> for example requests

Register a user from /register
Then login at /login

Send request to sequncer since session tokens look similar
Analyze now and overall randomness is poor

since the few charaters are just our username and then random 6 alphanumerals

/v1/recipes?q=

Leaking sensitive Data like ID's
/v1/data/titles reveals all ID's of recipe, intended behaviour no excessive data exposure

/v1/data/admin/recipes
We are unauthorized to use this endpoint since we are not admin

/v1/data/internal/uptime : only accessible internally so maybe we need SSRF to access this endpoint




