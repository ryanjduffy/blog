---
layout: post
title: Appending data to a document in Fauna
date: 2021-01-31 13:57 -0800
tags: faunadb
---

I've been diving into [faunadb](https://fauna.com/) recently. I found it via its integration with [Netlify](https://netlify.com/) and its GraphQL support. But, this isn't about either of those things!

One feature I've enjoyed with fauna is its support for [User-defined Functions (UDFs)](https://docs.fauna.com/fauna/current/api/fql/functions). I used to created stored procedures in my relational DB past and like being able to consolidate data-related logic on the database to keep that logic centralize and reduce round-trips from the service tier.

I've been building some custom functions to use as GraphQL endpoints to present a consistent, managed view of the data for my service tier / front-end. To support this, I created a quick utility UDF that merges an arbitrary object into the `data` payload of a document.

<!--more-->

The function, `addToData`, accepts a target object, `into`, and an object of additional data, `data`. The contents of the `data` variable is merged into the `data` key of the `into` object to produce the output. If I need to add a computed value to a document, this is a quick way to add that in a consistent manner.

```
CreateFunction({
	name: 'addToData',
	body: Query(
		Lambda(
			["into", "data"],
			Merge(
				Var("into"),
				{data: Var("data")},
				Lambda(
					["key", "a", "b"],
					If(
						Equals(Var("key"), "data"),
						Merge(Var("a"), Var("b")),
						Var("a")
					)
				)
			)
		)
	)
})
```

Example

```
Call(
	Function('addToData'),
	[
		Get(Ref(Collection("users"), "288896729057591808")),
		{
			stuff: 'okay'
		}
	]
)

>> results

{
	ref: Ref(Collection("users"), "288896729057591808"),
	ts: 1611772240815000,
	data: {
		name: "Ryan Duffy",
		stuff: "okay"
	}
}
```