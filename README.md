# ☎️ Phone

This test is a part of our hiring process at Aircall for the Frontend Engineer position. It should take you between 3 to 5 hours, depending on your experience, to implement the minimal version. But we thought about a few bonuses, so feel free to spend some time on them if you want.

*Feel free to apply! Drop us a line with your LinkedIn/GitHub/Twitter at jobs@aircall.io.*

## Context

Aircall is on a mission to revolutionize the business phone industry! This test is about (re) building a small part of Aircall’s main application. You’ll use dedicated APIs providing mocked data for that.

## Exercise

You’ll have to build an application, either with React, Angular, Vue or any other framework. We do write our applications in React but you can use other frameworks or libraries here.

This application must:
- display a paginated list of calls that you’ll retrieve from our backend
- if the user clicks on a call, display its details
- archive 1 or several calls
- group calls by date
- handle real time events

Bonus:
- use our own Design System UI library, called [tractor](http://tractor.aircall.io/) 🚜 Link to the NPM Repository [here](https://www.npmjs.com/package/@aircall/tractor).
- provide filtering feature, to filter calls  by type (archived, missed …)
- use Typescript
… and many others! Don’t be afraid to use new or unknown libraries, we’d love to learn new things!  

## APIs

We’ve built 2 APIs for this test, so you can choose between a REST API or a GraphQL API. Both expose the same data, so it’s really about which one you prefer.

### Model

Both APIs use the same models.

Call Model

```
type Call {
  id: ID! // "unique ID of call"
  direction: String! // "inbound" or "outbound" call
  from: String! // Caller's number
  to: String! // Callee's number
  duration: Float! // Duration of a call (in seconds)
  is_archived: Boolean! // Boolean that indicates if the call is archived or not
  call_type: String! // The type of the call, it can be a missed, answered or voicemail.
  via: String! // Aircall number used for the call.
  created_at: String! // When the call has been made.
  notes: Note[]! // Notes related to a given call
}
```

Note Model

```
type Note {
  id: ID!
  content: String!
}
```

### GraphQL API

Base URL: https://frontend-test-api.aircall.io/graphql

#### Authentication

You must first authenticate yourself before requesting the API. You can do so by executing the Login mutation. See below.

#### Queries

All the queries are protected by a middleware that checks if the user is authenticated with a valid JWT.

`paginatedCalls` returns a list of paginated calls. You can fetch the next page of calls by changing the values of `offset` and `limit` arguments.

```
paginatedCalls(
  offset: Float = 0
  limit: Float = 10
): PaginatedCalls!

type PaginatedCalls {
  nodes: [Call!]
  totalCount: Int!
  hasNextPage: Boolean!
}
```

`activitiy` return a single call if any, otherwise it returns null.

```
call(id: Float!): Call
```

`me` return the current authenticated user.

```
me: UserType!
```

```
type UserType {
  id: String!
  username: String!
}
```

#### Mutations

To be able to grab a valid JWT token, you need to execute the login mutation.

`login` receives the username and password as 1st parameter and return the access_token and the user identity.

```
login(input: LoginInput!): LoginType!

input LoginInput {
  username: String!
  password: String!
}
```

Once you are correctly authenticated you need to pass the Authorization header for all the next calls to the GraphQL API.

```
{
  "Authorization": "Bearer <YOUR_ACCESS_TOKEN>"
}
```

Note that the access_token is only available for 10 minutes. You need to ask for another fresh token by executing the same mutation.

`archiveCall` as the name implies it either archive or unarchive a given call.If the call doesn't exist, it'll throw an error.

```
archiveCall(id: ID!): Call!
```

`addNote` create a note and add it prepend it to the call's notes list.

```
addNote(input: AddNoteInput!): Call!

input AddNoteInput {
  activityId: ID!
  content: String!
}
```

#### Subscriptions

To be able to listen for the mutations/changes done on a given call, you can call the `onUpdateCall` using an actibity ID.

`onUpdateCall` receives the call ID as 1st parameter and returns a call instance.

```
onUpdateCall(id: ID): Call!
```

Now, whenever a call data changed either via the `addNote` or `archiveCall` mutations, you will receive a subscription event informing you of this change.

_Don't forget to pass the Authorization header with the right access token in order to be able to listen for these changes_

### REST API

Base URL: https://frontend-test-api.aircall.io

#### Authentication

You must first authenticate yourself before requesting the API. You can do so by sending a POST request to `/auth/login`. See below.

#### GET endpoints

All the endpoints are protected by a middleware that checks if the user is authenticated with a valid JWT.

`GET` `/calls` returns a list of paginated calls. You can fetch the next page of calls by changing the values of `offset` and `limit` arguments.

```
/calls?offset=<number>&limit=<number>
```

Response:
```
{
  nodes: [Call!]
  totalCount: Int!
  hasNextPage: Boolean!
}
```

`GET` `/calls/:id` return a single call if any, otherwise it returns null.

```
/calls/:id<uuid>
```

`GET` `/me` return the current authenticated user.

```
/me
```

Response
```
{
  id: String!
  username: String!
}
```

#### POST endpoints

To be able to grab a valid JWT token, you need to call the following endpoint:

`POST` `/auth/login` receives the username and password in the body and returns the access_token and the user identity.

```
/auth/login

// body
{
  username: String!
  password: String!
}
```

Once you are correctly authenticated you need to pass the Authorization header for all the next calls to the REST API.
```
{
  "Authorization": "Bearer <YOUR_ACCESS_TOKEN>"
}
```

Note that the access_token is only available for 10 minutes. You need to ask for another fresh token by calling the same previous endpoint.

`POST` `/calls/:id/note` create a note and add it prepend it to the call's notes list.

```
`/calls/:id/note`

Body
{
  content: String!
}
```

It returns the `Call` as response or an error if the note doesn't exist.

#### PUT endpoints

`PUT` `/calls/:id/archive` as the name implies it either archive or unarchive a given call.If the call doesn't exist, it'll throw an error.

```
PUT /calls/:id/archive
```

#### Real-time

In order to be aware of the changes done on a call, you need to subscribe to this private channel: `private-aircall` and listen for the following event: `update-call` which will return the call payload.

This event will be called each time you add a note or archive a call.

Note that, you need to use Pusher SDK in order to listen for this event.

Because this channel is private you need to authenticate first, to do that, you need to make `authEndpoint` point to: `<https://frontend-test-api.aircall.io/pusher/auth>`

#### Errors

The REST API can return different type of errors:

`400` `BAD_REQUEST` error, it happens when you provide some data which doesn't respect a given shape.

Example
```
{
  "statusCode": 400,
  "message": [
    "content must be a string",
    "content should not be empty"
  ],
  "error": "Bad Request"
}
```

`401` `UNAUTHORIZED` error, it happens when the user is not authorized to perform an action or if his token is no longer valid

Example
```
{
  "statusCode": 401,
  "message": "Unauthorized"
}
```

`404` `NOT_FOUND` error, it happens when the user request a resource that no longer exist.

Example
```
{
  "statusCode": 404,
  "message": "The call does not exist!",
  "error": "Not Found"
}
```

## Submission

We don’t provide any boilerplate as a simple [CRA](https://create-react-app.dev/) will be enough here. Feel free to create a repository and send us the link once you’re ready. If the repository is private, please invite @litil and @kamalbennani.

We'll try to review it in the next 48 hours and get back to you in order to talk about your code!

Contact us at jobs@aircall.io if you need more details.
