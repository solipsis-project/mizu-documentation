# Building an Image Gallery in Mizu

The important key concepts of an application running on Mizu are as follows:

- Users modify the state of the application by publishing messages. We call messages designed to modify the state of an application a Transaction.
- Application endpoints are Streams, which consist of a query into the database along with metadata on how to render that information.
- By writing queries that check for the presence of reserved fields (particularly `$schema` and `$signatures`), the application can enforce constraints on what a valid Transaction looks like.

Thus, when building an application on Mizu, or adding a feature to an application on Mizu, do the following:

- Consider what the possible Transactions are.
- Write a schema and query for each transaction.

We will explore a hypothetical image galley app built on Mizu, with each step building on the previous step by adding a useful feature. Each step will follow a similar thought process: outline the Transactions first, then write queries for them.

## Step 1: User Accounts

Image galleries typically require users to have an account in order to upload images. This allows the gallery admins to ban users that don't follow the gallery's rules. The set of users (and their permissions) can be controlled with a stream.

In this case, the user ids are public keys, and all of a user's transactions are signed with that key.

Right now we can imagine two different types of transactions: a "join" message that anyone can publish to create an account by posting their public key, and a "kick" to remove accounts, published by either the account owner or an admin. A kick overrides a join: once kicked, a user cannot rejoin.

We just created our own conflict-free replicated data type: this specific data type is commonly referred to as a "two-phase set".

We could implement this like so:

```
$GalleryUsersStream = echo '{
	"admin_key": "$ADMIN_KEY",
	
	"join_message": {
		{
			"@id": "?id"
			"stream": { "$ref": "#" },
			"$$signatues": {
				"key": "?user_key"
			}
			"action_type:" "join"
			"user_key": "?user_key",
			"user_name": "?user_name"
		},
	},
	"kick_message": {
		{
			"stream": "#",
			"$$signatues": {
				"@union": [
					{"key": "?user_key"},
					{"key": "$ADMIN_KEY" }
				]
			}
			"action_type:" "kick"
			"user_key": "?user_key",
		},
	}
	"get_users": {
		"@select" : ["?user_name", "?user_key"],
		"@where": [
			{ "$include": "#/join_message },
			{
				"@not": { "$include": "#/kick_message" }
			}
				
		]
	}
}' | mizu publish
```

## Step 2: User Uploaded Submissions

Once users are enrolled in the gallery, they can start uploading submissions.

We'll define two new action types: "upload_submission" and "redact_submission".

```
$GallerySubmissionsStream = echo '{
	"admin_key": "$ADMIN_KEY",
	"upload_submission": {
		"stream": { "$ref": "#" },
		"$$signatures": {
			"key": "?user_key"
		}
		"user_key": "?user_key",
		"action_type": "upload_submission"
		"content": "?content",
		"allow_comments": "?allow_comments"
	}
	"redact_submission": {
		"stream": { "$ref": "#" },
		"$$signatures": {
			"@union": [
				{"key": "?user_key"},
				{"key": "$ADMIN_KEY"}
			]
		}
		"user_key": "?user_key",
		"action_type": "redact_submission"
		"submission": "?submission"
	}
	"get_user_submissions": {
		"@select" : "?submission",
		"@where": [
			{ "@bind": { "?user_name": { "$var": "user_name" } } },
			{ "$include": "https://mizu.stream/message/$GalleryUsersStream/get_users/@where" },
			},
			{
				"$include": "#upload_submission,
				"@id": "?submission"
			},
			{
				"@not": { "$include": "#redact_submission" }
			}		
		]
	}
}
```

"get_user_submissions" is a query that returns all submissions by a given user, using the "$var" special field to allow users to specify the user name. So we might use a URI like:

https://mizu.stream/query/$GALLERY_STREAM/get_user_submissions?user_name=hunter2

Which would resolve to a list of all submissions submitted by user "hunter2" (and not subsequently redacted.)

One consequence of this design is that it's possible to have multiple users with the same name. (Messages can't cause a conflict, after all, even join messages with the same name.) Under this system, users are distinguished by their public key instead. In that case, the above could return multiple sets of results.

This introduces a possible attack vector: a bad actor could overwhelm the network with lots of accounts with the same name, and those fake users will show up in search results until an admin kicks them. A conventional website would have rate-limiting in place, but in Mizu, no such rate-limiting can exist. So how do we get around this?

One approach would be for there to be a list of "verified" accounts that can only be modified by an admin. Creating an account does not cause it to be immediately verified.

This only requires a single new action type: "verify_account". (Accounts don't get unverified, they just get kicked instead.)

```
$GalleryVerifiedAccounts = echo '{
	"admin_key": "$ADMIN_KEY",
	"verify_account": {
		"stream": { "$ref": "#" },
		"$$signatures": {
			"key": "$ADMIN_KEY"
		}
		"user_key": "?user_key",
		"user_name": "?user_name",
		"action_type": "verify_account"
	}
	"get_users": {
		"@select" : ["?user_name", "?user_key"],
		"@where": [
			{ "$include": "https://mizu.stream/message/$GalleryUsersStream/get_users/@where" },
			{ "$include": "#/verify_account },
		]
	}
	"get_verified_user_submissions": {
		"@select" : "?submission",
		"@where": [
			{ "$include": "https://mizu.stream/message/$GallerySubmissionsStream/get_user_submissions/@where" },
			{ "$include": "#/verify_account },	
		]
	}
}
```

# TODO: Finish this example

## Step 3: User Blocking

## Step 4: Comments

Another feature that web galleries have is comments. Let's enable submissions to have comments. 

We'll create new action types: "post_comment" and "redact_comment". A comment can be redacted by the user who made it, the owner of the submission, or an admin.

Comments can also be edited.

## Step 5: Favorites

Users can also have "favorites" folders. These folders can be public (can be queried by other users) or private (can only be queried by their owner). Either way, the structure of these folders are the same.