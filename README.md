# Client Library API

This API describes interacting with GroupMe via our iOS and Android SDKs.

__Note:__ This is a work in progress and can change at any time!

# Terms

* __Client__ - The application or service integrated with GroupMe
* __Client ID__ - The key publicly identifying your client
* __Client Secret__ - The token secretly identifying your client
* __Access Token__ - The token that authenticates a user

# Authentication

The GroupMe client library uses [OAuth 2.0](http://oauth.net/2/) for
authentication.

All requests must be made over __HTTPS__.

## Phone-based applications

### Obtain an access token

Use the __client_credentials__ grant type to get a token.

    POST https://api.groupme.com/clients/tokens
      ?client_id=YOUR_CLIENT_ID
      &client_secret=YOUR_CLIENT_SECRET
      &device_id=YOUR_DEVICE_ID
      &phone_number=YOUR_PHONE_NUMBER
      &grant_type=client_credentials

### Verified phone number

If your phone number has been verified with GroupMe, we respond with:

    HTTP/1.1 201 Created
    { access_token : YOUR_ACCESS_TOKEN, user_id : USER_ID, user_name: USER_NAME }


### Unverified phone number

If your phone number has NOT been verified with GroupMe, we respond with:

    HTTP/1.1 401 Unauthorized

A __text will be sent__ to the user containing a short __authorization code__.

You will need to re-request that token using the __authorization_code__ grant
type.

    POST https://api.groupme.com/clients/tokens
      ?client_id=YOUR_CLIENT_ID
      &client_secret=YOUR_CLIENT_SECRET
      &device_id=YOUR_DEVICE_ID
      &phone_number=YOUR_PHONE_NUMBER
      &grant_type=authorization_code
      &code=YOUR_AUTHORIZATION_CODE

If the code is correct, we respond with:

    HTTP/1.1 201 Created
    { access_token : YOUR_ACCESS_TOKEN, user_id : USER_ID }

# Making Requests

You can request resources using your access token:

    GET https://api.groupme.com/clients/groups
      ?client_id=YOUR_CLIENT_ID
      &token=YOUR_ACCESS_TOKEN

Most endpoints require authentication. We indicate which do not require your
token to be included in the request.

# Getting Responses

All responses look something like this:

    {
      meta : {
        code : 200
      },
      notifications : ...
      response : ...
    }

## Response

This is the object (or objects) you are manipulating.

## Meta

This section is intended to provide additional information to developers.

### Code

For clients that cannot read HTTP status codes, we provide it directly in the
response.

### Errors

If you receive 400 Bad Request, you can expect errors in the response:

    HTTP/1.1 400 Bad Request
    {
      meta : {
        code: 400,
        errors: [
          "Group needs a topic",
          "You can only add 25 people to a group"
        ]
      },
      response: null
    }

If you receive 401 Unauthorized:

    HTTP/1.1 401 Unauthorized
    {
      meta : {
        code: 401,
        errors: [
          "An access token is required"
        ]
      },
      response: null
    }

If you receive 403 Forbidden:

    HTTP/1.1 403 Forbidden
    {
      meta : {
        code: 403,
        errors: [
          "You can't modify a group you aren't in"
        ]
      },
      response: null
    }

## Notifications

This is information directed at users.

# Endpoints

* __ID__ - All resource IDs are strings (do not assume they are numbers or in any order)
* __Timestamps__ - All timestamps are expressed epoch integers
* __Format__ - Everything is JSON unless specified otherwise

## Group

<table>
  <tr>
    <th>id</th>
    <td>A unique identifier for this group</td>
  </tr>
  <tr>
    <th>creator_id</th>
    <td>The id of the user who created this group</td>
  </tr>
  <tr>
    <th>phone_number</th>
    <td>The phone number of the group</td>
  </tr>
  <tr>
    <th>created_at</th>
    <td>When the group was created</td>
  </tr>
  <tr>
    <th>updated_at</th>
    <td>When the group was last updated</td>
  </tr>
  <tr>
    <th>size</th>
    <td>How many <b>active</b> users are in the group (See <b>Memberships</b>)</td>
  </tr>
  <tr>
    <th>topic</th>
    <td>The human-readable identifier of the group</td>
  </tr>
  <tr>
    <th>description</th>
    <td>A short description of who is in the group</td>
  </tr>
  <tr>
    <th>last_line</th>
    <td>
      Minimal data about the last chat line sent to the group
      <br />
      The goal is to provide enough data to richly display a group preview
      without making additional calls to lines, members, users, etc.
    </td>
  </tr>
</table>


### Groups

Get a list of all your groups.

<table>
  <tr>
    <th>URL</th>
    <td>https://api.groupme.com/clients/groups</td>
  </tr>
  <tr>
    <th>Method</th>
    <td>GET</td>
  </tr>
  <tr>
    <th>Requires Auth</th>
    <td>Yes</td>
  </tr>
</table>

#### Request

    GET https://api.groupme.com/clients/groups
      ?client_id=YOUR_CLIENT_ID
      &token=YOUR_ACCESS_TOKEN

#### Response

    HTTP/1.1 200 OK
    {
      groups : [
        {
          id              : "1234567890",
          creator_id      : "1234567890",
          phone_number    : "+1 9174444444",
          created_at      : 1302623328,
          updated_at      : 1302623328,
          size            : 5,
          topic           : "Night Out",
          description     : "Bob, Anne, and 3 others",
          last_line       : {
            id          : "1234567890",
            user_id     : "1234567890",
            avatar_url  : "http://example.com/bob.png",
            name        : "Bob",
            text        : "Hello World",
            created_at  : 1302623328
          }
        },
        ...
      ]
    }

### Create

Create a new group.

<table>
  <tr>
    <th>URL</th>
    <td>https://api.groupme.com/clients/groups</td>
  </tr>
  <tr>
    <th>Method</th>
    <td>POST</td>
  </tr>
  <tr>
    <th>Requires Auth</th>
    <td>Yes</td>
  </tr>
</table>

#### Group Params

<table>
  <tr>
    <th>topic</th>
    <td>How members identify this group (e.g. Family, Night Out, etc.)</td>
  </tr>
  <tr>
    <th>memberships</th>
    <td>
        A minimal list of people you want in the group.
    </td>
  </tr>
</table>

#### Membership Params

You must provide a name and exactly __one other__ form of identification.

This can be (in order of strength):

* __`user_id`__ - The GroupMe unique identifier for the user
* __`phone_number`__ - The user's phone number with country code
* __`email`__ - The user will receive an email invite

<table>
  <tr>
    <th>name</th>
    <td>What the user will be called in the group</td>
  </tr>
  <tr>
    <th>identifier</th>
    <td>
        (See above list)
    </td>
  </tr>
</table>


#### Request

    POST https://api.groupme.com/clients/groups
      ?client_id=YOUR_CLIENT_ID
      &token=YOUR_ACCESS_TOKEN
    {
      group : {
        topic       : "Night Out",
        memberships : [
          {
            name          : "Bob",
            phone_number  : "+1 2125555555"
          },
          {
            name          : "Anne",
            email         : "anne@example.com"
          },
          {
            name          : "John",
            user_id       : "1234567892"
          }
        ]
      }
    }

#### Response

    HTTP/1.1 201 Created
    {
      group : {
        id              : "1234567890",
        creator_id      : "1234567890",
        phone_number    : "+1 9174444444",
        created_at      : 1302623328,
        updated_at      : 1302623328,
        size            : 5,
        topic           : "Night Out",
        description     : "Bob, Anne, and 3 others",
        last_line       : null
      }
    }

### Destroy

Create a new group.

<table>
  <tr>
    <th>URL</th>
    <td>https://api.groupme.com/clients/groups/GROUP_ID</td>
  </tr>
  <tr>
    <th>Method</th>
    <td>DELETE</td>
  </tr>
  <tr>
    <th>Requires Auth</th>
    <td>Yes</td>
  </tr>
</table>

#### Request

  DELETE https://api.groupme.com/clients/groups/GROUP_ID
    ?client_id=YOUR_CLIENT_ID
    &token=YOUR_ACCESS_TOKEN

#### Response

  HTTP/1.1 200 OK

## Membership

A membership is the resource that __links users to groups__.

It contains meta-data about the user's relationship to the group.

<table>
  <tr>
    <th>id</th>
    <td>A unique identifier for this membership.</td>
  </tr>
  <tr>
    <th>user_id</th>
    <td>A unique identifier for the user.</td>
  </tr>
  <tr>
    <th>name</th>
    <td>What the user is called in this group only</td>
  </tr>
  <tr>
    <th>avatar_url</th>
    <td>The URL to the user's avatar</td>
  </td>
  <tr>
    <th>state</th>
    <td>The state of the membership (e.g. pending, active, muted, removed, exited)</td>
  </tr>
  <tr>
    <th>created_at</th>
    <td>When the membership was created</td>
  </tr>
  <tr>
    <th>updated_at</th>
    <td>When the membership was last updated</td>
  </tr>
</table>

### States

#### Pending

The user is __unreachable__ and will __NOT__ receive messages.

Since a membership accepts non-phone number identifiers like email, we need
the user to complete an invite process before they can be reached via SMS or
push.

This means providing their phone number for SMS and installing the app for
push.

If we do not support SMS in the user's country, they are considered
__unreachable__ and therefore pending.

#### Active

The user is __reachable__ and will receive messages.

#### Muted

The user is __reachable__ but will __NOT__ receive messages.

The user is still part of the group, but will not be notified in real-time.

#### Exited

The user has left the group.

The user has the option to __re-join__ the group by sending a message to it.

#### Removed

Someone from the group __removed__ the user.

The user __CANNOT re-join__ the group. They may be __re-added__ by an existing
group member.


### Memberships

Get a list of all memberships for a group.

<table>
  <tr>
    <th>URL</th>
    <td>https://api.groupme.com/clients/groups/GROUP_ID/memberships</td>
  </tr>
  <tr>
    <th>Method</th>
    <td>GET</td>
  </tr>
  <tr>
    <th>Requires Auth</th>
    <td>Yes</td>
  </tr>
</table>

#### Request

    GET https://api.groupme.com/clients/groups/GROUP_ID/memberships
      ?client_id=YOUR_CLIENT_ID
      &token=YOUR_ACCESS_TOKEN

#### Response

    HTTP/1.1 200 OK
    {
      memberships : [
        {
          id          : "1234567890",
          user_id     : "1234567890",
          name        : "Bob",
          avatar_url  : "http://example.com/bob.png",
          state       : "active",
          created_at  : 1302623328,
          updated_at  : 1302623328
        },
        {
          id          : "1234567891",
          user_id     : "1234567891",
          name        : "Anne",
          avatar_url  : "http://example.com/anne.png",
          state       : "pending",
          created_at  : 1302623328,
          updated_at  : 1302623328
        },
        {
          id          : "1234567892",
          user_id     : "1234567892",
          name        : "John",
          avatar_url  : null,
          state       : "muted",
          created_at  : 1302623328,
          updated_at  : 1302623328
        },
        {
          id          : "1234567893",
          user_id     : "1234567893",
          name        : "Reginald",
          avatar_url  : "http://example.com/reginald.png",
          state       : "exited",
          created_at  : 1302623328,
          updated_at  : 1302623328
        },
        {
          id          : "1234567894",
          user_id     : "1234567894",
          name        : "Watson",
          avatar_url  : "http://example.com/watson.png",
          state       : "removed",
          created_at  : 1302623328,
          updated_at  : 1302623328
        },
        ...
      ]
    }

### Create

Add a user to the group.

<table>
  <tr>
    <th>URL</th>
    <td>https://api.groupme.com/clients/groups/GROUP_ID/memberships</td>
  </tr>
  <tr>
    <th>Method</th>
    <td>POST</td>
  </tr>
  <tr>
    <th>Requires Auth</th>
    <td>Yes</td>
  </tr>
</table>

#### Request (via ID)

    POST https://api.groupme.com/clients/groups/GROUP_ID/memberships
      ?client_id=YOUR_CLIENT_ID
      &token=YOUR_ACCESS_TOKEN
    {
      membership : {
        name    : "John",
        user_id : "1234567891"
      }
    }

#### Response

    HTTP/1.1 201 Created
    {
      membership : {
        id          : "1234567891",
        user_id     : "1234567891",
        name        : "John",
        avatar_url  : null,
        state       : "active",
        created_at  : 1302623328,
        updated_at  : 1302623328
      }
    }

#### Response (only email specified)

If an email is specified and the user does not exist, then an email invite is sent and the membership response is slightly different:

    HTTP/1.1 201 Created
    {
      membership : {
        id          : "invite-1234567891",
        user_id     : null,
        name        : "John",
        avatar_url  : null,
        state       : "pending",
        created_at  : 1302623328,
        updated_at  : 1302623328
      }
    }

### Destroy

Remove a member from/leave the group.

If the requesting user is the user being removed, the membership is considered
__exited__.

If the requesting user is __NOT__ the user being removed, the membership is
considered __removed__.

#### Request

    DELETE https://api.groupme.com/clients/groups/GROUP_ID/memberships/SOME_MEMBERSHIP_ID
      ?client_id=YOUR_CLIENT_ID
      &token=YOUR_ACCESS_TOKEN

#### Response

    HTTP/1.1 200 OK

## Line

<table>
  <tr>
    <th>id</th>
    <td>A unique identifier for this line</td>
  </tr>
  <tr>
    <th>source_guid</th>
    <td>
      A unique identifier for this line sent by the client.
      <br />
      This is useful for duplicate detection on clients.
    </td>
  </tr>
  <tr>
    <th>created_at</th>
    <td>When the line was created</td>
  </tr>
  <tr>
    <th>user_id</th>
    <td>The id of the user who created this line</td>
  </tr>
  <tr>
    <th>group_id</th>
    <td>The id of the group that this line was created in</td>
  </tr>
  <tr>
    <th>avatar_url</th>
    <td>The url to the current avatar of the user who created the line</td>
  </tr>
  <tr>
    <th>name</th>
    <td>The name of the user who created this line at the time of creation</td>
  </tr>
  <tr>
    <th>text</th>
    <td>The text of the line</td>
  </tr>
  <tr>
    <th>picture_url</th>
    <td>The url of the attached picture</td>
  </tr>
  <tr>
    <th>location</th>
    <td>The attached location</td>
  </tr>
</table>


### List

<table>
  <tr>
    <th>URL</th>
    <td>https://api.groupme.com/clients/groups/GROUP_ID/lines</td>
  </tr>
  <tr>
    <th>Method</th>
    <td>GET</td>
  </tr>
  <tr>
    <th>Requires Auth</th>
    <td>Yes</td>
  </tr>
</table>

#### Request

    GET https://api.groupme.com/clients/groups/GROUP_ID/lines
      ?client_id=YOUR_CLIENT_ID
      &token=YOUR_ACCESS_TOKEN
      
#### Response

    HTTP/1.1 200 OK
    {
      "lines" : [
        {
          "id"          : "1234567890",
          "source_guid" : "YOUR_GUID_HERE",
          "created_at"  : 1302623328,
          "user_id"     : "1234567890",
          "group_id"    : "1234567890",
          "avatar_url"  : "http://example.com/anne.png"
          "name"        : "Anne",
          "text"        : "Hello world",
          "picture_url" : null,
          "location"    : {
            "lat"                   : "40.738206",
            "lng"                   : "-73.993285",
            "name"                  : "GroupMe HQ",
            "foursquare_venue_id"   : "1234567890",
            "foursquare_checkin"    : true
          },
        ...
      ]
    }

### Create

Add a line to the group.

<table>
  <tr>
    <th>URL</th>
    <td>https://api.groupme.com/clients/groups/GROUP_ID/lines</td>
  </tr>
  <tr>
    <th>Method</th>
    <td>POST</td>
  </tr>
  <tr>
    <th>Requires Auth</th>
    <td>Yes</td>
  </tr>
</table>


#### Request

    POST https://api.groupme.com/clients/groups/GROUP_ID/lines
      ?client_id=YOUR_CLIENT_ID
      &token=YOUR_ACCESS_TOKEN
    {
      line : {
        source_guid : "YOUR_GUID_HERE",
        text        : "Hello world",
        picture_url : null,
        location    : {
          lat                    : "40.738206",
          lng                    : "-73.993285",
          name                   : "GroupMe HQ",
          foursquare_venue_id    : "1234567890",
          foursquare_checkin     : true
        }
      }
    }

#### Response

    HTTP/1.1 201 Created
    {
      line : {
        id          : "1234567890",
        source_guid : "YOUR_GUID_HERE",
        created_at  : 1302623328,
        user_id     : "1234567890",
        group_id    : "1234567890",
        avatar_url  : "http://example.com/anne.png"
        name        : "Anne",
        text        : "Hello world",
        picture_url : null,
        location    : {
          lat                   : "40.738206",
          lng                   : "-73.993285",
          name                  : "GroupMe HQ",
          foursquare_venue_id   : "1234567890",
          foursquare_checkin    : true
        }
      }
    }


## Conferences

Conferences call each member of the group to start a conference call.

<table>
  <tr>
    <th>URL</th>
    <td>https://api.groupme.com/clients/groups/GROUP_ID/conferences</td>
  </tr>
  <tr>
    <th>Method</th>
    <td>POST</td>
  </tr>
  <tr>
    <th>Requires Auth</th>
    <td>Yes</td>
  </tr>
</table>

#### Request

    POST https://api.groupme.com/clients/groups/GROUP_ID/conferences
      ?client_id=YOUR_CLIENT_ID
      &token=YOUR_ACCESS_TOKEN

#### Response

    HTTP/1.1 201 Created

## Users

User names can be updated as follows:

<table>
  <tr>
    <th>URL</th>
    <td>https://api.groupme.com/clients/users/USER_ID/</td>
  </tr>
  <tr>
    <th>Method</th>
    <td>PUT</td>
  </tr>
  <tr>
    <th>Requires Auth</th>
    <td>Yes</td>
  </tr>
</table>

#### Request

    PUT https://api.groupme.com/clients/users/USER_ID
      ?client_id=YOUR_CLIENT_ID
      &token=YOUR_ACCESS_TOKEN
      &user[name]=NEW_USER_NAME

#### Response

    HTTP/1.1 200 OK

    {
      user: {
        id: USER_ID,
        name: USER_NAME
      }
    }


# TODO

None as of now #)
