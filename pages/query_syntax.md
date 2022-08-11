# Query Syntax

TODO

The following table shows equivalent examples of json-rql syntax and the corresponding SPARQL syntax, incorporating Mizu features such as embedded request parameters:

- "select ?message where ?message <stream> <${#}> ; <$signatures> ?sig . ?sig <key> <$KEY>"
- '{ "@select": "?message",
  		"@where": {
  			"@id": "?message",
  			"stream": { "$ref": "#" },
  			"$$signatures": {
  				"key": "$KEY",
  			}
  		}
	}'