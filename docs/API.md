# General Rules / Slight Oddities #

* Time is expressed in seconds since the UNIX epoch. Use a library to convert to whatever datetime system you need.
* There is a confusingly but necessarily, a concept of CSH users AND a concept of in-game users; many people who don't have a CSH account play NetHack, for instance (past members, friends, coworkers, family, freshmen without accounts yet) and there is also a much smaller group of people who have a CSH account but have their in-game username as something different. If you get back a username from requests inside the `app/` tree of API requests, you would do well to assume these usernames are only valid for that game.
* ID numbers are all unique within whatever concept you're in (i.e. all achievement IDs are unique, even across applications, but they would still step on the ID numbers of, say, users, which is a completely unrelated concept), and can be used as such if you require unique concept-global IDs inside your application.

# Unauthenticated Read Operations #

## GET /apps ##
Returns an `apps` list containing the `name` and `description` of applications that have achievements.

Example output:
````
{"apps":[{"name":"Nethack","description":"A text-based, turn-based, dungeon-crawling RPG"},{"name":"CSH GameJam 2011","description":"Computer Science House GameJam Winter 2011"},{"name":"netrek","description":"Paradise Netrek @ whitestar.cs.rit.edu"}]}
````

## GET /apps/:appName ##
Returns a list of `achievements` that are contained within this application.

* `id` = A guaranteed-unique number ID number that refers to this achievement. This ID number is global to all achievements, not just achievements for a particular application. This is mostly included for backwards compatibility with existing achievements using applications, which previously had to specify know this number for updates.
* `title` = A small (<=80 characters, mostly much less) witty title for this achievement.
* `description` = A <=255 character description of the achievement.
* `score` = How much this achievement is worth when completed.

Example output:
````
GET /apps/nethack
{"achievements":[{"id":1,"title":"Titillating Tech Talk","description":"Chat with Andy Potter","score":5},{"id":2,"title":"Neither Rain nor Sleet nor Snow","description":"Recieve a scroll from the mail daemon","score":5}]}
````

## Get /apps/:appName/events ##
Returns a list of `events` (aka 'binges') that this application is currently having or has had. You could use this information to create some sort of live scoreboard, for instance, or for calendar / dropdown purposes.

Time is expressed in seconds since the UNIX epoch.

Example output:
````
GET /apps/nethack/events
{"events":[{"app name":"Nethack","event title":"Independence Day Full Moon","start time":1341324000,"end time":1341367200},{"app name":"Nethack","event title":"Winter Quarter New Moon","start time":1357992000,"end time":1358078400}]}
````

## GET /apps/:appName/users ##
Returns a list of `users` that have achievement progress in this application.

````
GET /apps/nethack/users
{"users":[{"username":"clockfort"},{"username":"russ"}]}
````

## GET /apps/:appName/users/:username ##

Returns a list of achievements that this user has in this application.

The user "has" the achievement if user progress = max progress. I'll probably add a "completed" tag to make things easier for API users at some point.


````
{"achievements":[{"app name":"Nethack","title":"Developers, Developers, Developers, Developers","description":"Make a non-trivial source commit","max progress":1,"user progress":1,"score":10,"updated at":1315633538}]}
````

## GET /users ##

Get a global list of `users` (which just contain `username`s).

## GET /events ##

Get a global list of `events`.

## GET /users/:userName ##

Get a list of all a user's achievments, in all applications.


# Write Operations #

These operations require an `appKey` to be passed in. This key is only valid for a single application; i.e. NetHack administrators/apps could not issue achievements for Netrek.

## POST /app/:appName/achievement ##

You must provide either an achievement title or ID number, and a username.

Simplest case bare-minimum example:
````
POST /app/:netHack/achievement
{ "appKey":"ccefe01e6e15c6dd076be4be3536fc84", "achievement":{"id":23, "username":"clockfort"} }
````

More complicated example (reference by title, possible multi-part achievement progress tracking, backdating achievements)
````
POST /app/:netHack/achievement
{ "appKey":"ccefe01e6e15c6dd076be4be3536fc84", "achievement":{"title":"Developers, Developers, Developers, Developers", "username":"clockfort", "user progress":1,"updated at":1315633538} }
````
