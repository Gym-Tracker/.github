## Tech Stack

### Database

- PostgreSQL
- Amazon RDS

### Web server

- Java Spring
- Amazon EC2

### Web app

- Typescript
- React
- Amazon Amplify

## Requirements

### Paint points

- Manually entering all 5-3-1 weights into Hevy every week

### Requirements

- Enter 1 rep max for squat, bench, deadlift and overhead press
- Have training max calculated
- Have weights calculated for a cycle of 3 sessions per week for 3 weeks
- Track which week you are on
- Store workouts using [[Backend Database]]
- Types of sets
	- Normal
	- Warm up
	- Failure
	- Drop set

### Use cases

#### New user

- Open app
- Make account
	- Enter username, email and password
- Enter 1 rep max for squat, bench, deadlift and overhead press
	- Program then calculates training maxes
- Start on week 1 workout 1
- Click start workout
- Page comes up listing the exercises and sets to do, with ticks next to them
	- Sets are filled out according to the programme but can be edited (if you edit the value and then delete the value it resets back to default)
- Do an exercise and tick it
- Timer begins?
	- Can skip the timer
- Once all exercises are done, click finish workout
- Workout is saved

#### Existing user

- Open app
- Home screen shows workout history and what the next workout will be
- Click start workout
- Do all the exercises
- Click finish work out

### Product Backlog

- Creating account
- Logging in
- Entering 1 rep maxes and calculation of training maxes for squat, bench, deadlift and overhead press
- Start workout button
- Workout screen with pre filled out sections
	- Sections for each exercise with sets
	- Sets have a type, weight and reps count
- Workout history on home screen

#### Must

#### Should

#### Could

#### Won't

## Development Log

- Requirements analysis ([[Requirements]])
- Decided on type of database - SQL or NoSQL
- Picked [[PostgreSQL]]
- Designed schema
- Looked into options for [[Backend Database]]
- Created Amazon RDS Postgres database

---

- Server can then run on EC2
- How to connect server to front-end?
	- 3 ways
		1. Keep the front and back end together
			- Server responds with HTML
			- Will need API for any decent interaction, like inputting data to be stored
		2.  API - separates front and back end
			- Flexibility for different platforms in the future
			- Allows for different possible front ends - web apps or native apps
			- If front end is a web app you use another server to host those static files
		3. Serverless functions
			- No server, just separate functions sitting on a platform like AWS
- Going to go with an API
- How to build API in Java?
- Looks like Spring is the way to go 
	- [ ] https://spring.io/guides/tutorials/rest/
	- [x] https://spring.io/guides/gs/rest-service/

---

- Start with front end
- Can't build an iOS app without a MAC - will have to build a web app
- Chose CSS modules for CSS
	- Makes CSS locally scoped for components and autocomplete for classes
	- Allows for code completion for entering CSS class names

---

- Got Spring working with requests and connected to postgres
- Create tables in postgres database
- Connect front end to spring backend https://www.freecodecamp.org/news/javascript-post-request-how-to-send-an-http-post-request-in-js/ 

---

- To fix CORS errors I had to add `@CrossOrigin(origins = "http://127.0.0.1:5173")` to the `WorkoutController`

---

- Should exercises be purely server side? or store the relationships between IDs and names on the client?

---

- Implemented getting workout objects from server by looping through each workout id, then looping through all sets (connected to a user) for each workout id and getting all of the sets grouped into exercises and putting all of that into workout objects for each workout
	- This is clunky but was done to avoid sending a query for every workout, to get its sets
- Need to now put these objects onto the UI
	- Some trouble with dates being returned from server as string but could be parsed into Date objects on the client

---

- Need to make changes to workout reflect in object

---

- Created a version of the Workout object type that included a flag for whether a set had been completed
- Could have made the `done` property of a set optional but I wanted to create an entirely separated type for an active workout to make the code easier to understand

---

- Found a repository with a big JSON file containing over 800 exercises and details about them
- Should this be bundled with the app, or in the database?
- Going to try just bundling the whole JSON file with the app to see how that goes
- Seems to have worked quite well
- Need to figure out how selecting a workout will work though...
	- Should exercise IDs be changed to the IDs used in the JSON file? i.e. strings not integers
	- This would require a change to the database and to the types in the front end code
	- Is there actually any reason to use integer IDs over strings considering I really doubt there will be any kind of performance bottleneck?
	- This would also mean that the workout table in the database is no longer needed
	- hmmmm...
	- How do you fetch an item from JSON given an ID? does that just require looping through the whole thing? or could a hashmap be hardcoded from it?
	- Turns out it is a little slow, makes the app stutter, but still perfectly functional - will optimise later
- Going to change the ID of every to an INT
	- Seems more fun than boringly changing a bunch of my current code
	- Curious what command will do it
	- ...feels more right to use an integer as an ID?
- Was going to try and do this with jq, then with bash, but finally couldn't figure out how to do it with those so did it with python
- WOOO got this all sorted - had to move some state management up to the App.tsx file but now you can add exercises to the active workout

---

*STRANGE CHARACTERS* discovered in the JSON file created by my python script!
What happened?

- Decided to change all of the IDs in the free-exercise-db JSON file to integer IDs
- Used a python script to parse the JSON from the file, loop though the list of objects, change each object's ID to an incrementing number, and then write this new list to a file
- ...time passed...
- Decided to edit all of the individual JSON files in the free-exercise-db repository (using a bash script I'm fairly proud of) - for my fork of it
- Created a new JSON file from those using one of the scripts in the repository
- Out of curiosity, decided to run a `diff` between the newly generated JSON file and the one in my project, made by the python script...

QUITE A FEW CHANGES???

- Looking at the differences, firstly the python generated file had `\u00c2\u00be` in the place of `Â¾`, bizarre - both look wrong?
- After looking online and checking the charset for both files, it seems this is a text encoding issue
- The python generated file is ASCII and the file from the repo is UTF-8
- Apparently the characters `Â¾` are the result when an UTF-8 multi-byte string is interpreted with a single-byte encoding like [ISO 8859-1](http://en.wikipedia.org/wiki/ISO/IEC_8859-1) or [Windows-1252](http://en.wikipedia.org/wiki/Windows-1252).
- [Encoding Problem: ISO-8859-1 vs Windows-1252](https://www.i18nqa.com/debug/bug-iso8859-1-vs-windows-1252.html)
- Looked into the original separated JSON files and found these weird characters were there from the start
- I think that somewhere in the process of creating the original, separated JSON files, UTF-8 characters got interpreted with some single-byte encoding and then saved back into UTF-8
- Looked back at the original data this was created from and found the weird characters there too! I think that these were scraped from the internet or something and that's where the weird interpretation happened

+ The rest of the differences are just down to different ordering of the elements, possibly because of the numerical
	+ NOPE, the order is different because the original is ordered by file name but without the file extension taken into account

Reading to do after this:
[The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!) – Joel on Software](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/)

Ended up contributing to free-exercise-db!

---

Need to do registering and logging in now

- Need to implement user sessions on the server
	- When a user logs in they are given a cookie by the server
	- The server knows that this cookie belongs to that user
	- When the user sends a request, they use their cookie to tell the server who they are
- Spring Session

+ Should it require a username and password or email and password?
	+ Username - bit more privacy? some people might not like giving over their email and it means I'm handling sensitive information
	+ Email - so that if people forget their password it can be reset through the email
+ Think email would be better - means I can implement "forgot password" later

- Looking into session management for the Java server
- Spring Session seems fairly complicated and kind of overkill for what I'm doing - meant for big scalability
- Java seems to have a way built in called `HttpSession`
- This would store the sessions in memory
	- Not very efficient apparently 
- Wait, I can just store them in the postgres database...
- Could make the server keep a cache in memory of the sessions to reduce the number of server requests...maybe an optimisation for later

+ Ok so if I store the sessions in postgres I need a way to delete stale session IDs
+ Can use Spring to schedule a method to check when each session was last used and if it exceeds a set time then remove it
+ On the client side, if a session ID doesn't work then delete that cookie
+ So need to keep updating when a session ID was last used
	+ Little potential optimisation - only record the day the session ID was last used and when accessing the sessions table to get a user id, compare the current day and the day from the result set of that query, only send an update query if the day is different (so only one query a day max instead of updating with *every* query)

- Companies seem to set client cookies to expire after a year?
- They must renew somehow though?

+ Decided to start using postman for testing requests since using curl is getting quite annoying - I've used it before, dunno why I didn't think of using it
+ Woahh cookies get sent automatically with every request
+ Can use `HttpServletRequest` to get cookies but going to use `@CookieValue` annotation for less code

- Added all this cookie and session stuff to the back end but now need to handle it all on the front end with a login/register screen
- Login/register screen that gets bypassed if the client has a valid session id

---

- Since I'm starting to add more requests in the front-end code, need to add a better way of constructing the URLs than fully hard coding them, need to store the base URL somewhere
- Could put the base URL in a variable in a `.ts` file and import it in each file there's a request
- Seems a better way to do it is to use environment variables, specifically a `.env` file
	- Means you can have multiple `.env` files for when the code is local or in production
	- Vite handles it quite well [Env Variables and Modes | Vite](https://vitejs.dev/guide/env-and-mode.html)

---

- Cookies not being set on the browser for some reason...
	- Had to add `credentials: "include"` to every fetch request in front end
	- Had to add `allowCredentials = "true"` to the `@CrossOrigin` annotation in the server
- Now implemented logging in with a session id cookie (if there is one) and logging out!
- I think I should set some things with the cookies like expiration and http only?
- Need to implement registering now and then I think the app will be technically usable

---

- Just added switching between the registering and logging in pages and allowing for actually creating an account
- Did a quick run through of the use cases and apart from the 5/3/1 stuff the app is, if clunky, technically usable!!!

---

- The login/register screen is the first thing people will interact with so needs to be decent
- Stopped the app leaving the register screen whatever you entered
- Added a bunch of basic stuff like messages for when details are incorrect/in use

+ Slightly unrelated but really should be hashing the passwords
+ Seems you generally hash passwords into a byte array
+ But this can be represented as a string by encoding it in base 64 or base 16 (hex)
+ But actually seems like it's best to just store as byte array (bytea) in postgres

---

- Decided to remove the exercise table and change exercise IDs in the set table to correspond with their IDs in the free-exercise-db
- Trying to give all exercises a numeric ID was too much hassle and this way I can very easily stay up to date if there are any contributions to that database since I won't need to maintain my own customised version ^d12cda

---

- Internet stopped working so I couldn't connect to the AWS postgres database
- Good actually, forced me to get the entire stack working locally for when there's a production version on AWS and the local version will be for development
- Working on different environments in Spring
- Figured out how to activate profiles in Spring which change the environment variables that are used - GREAT

---

- Changed my mind on [[#^d12cda|exercise IDs]] and will be using numbers or some other identifier instead
- This is because I want to be able to change the names of the exercises over time (they currently have names I don't like) so I don't want the ID to be tied to their name
- Moved to the original exercises.json repository because it had a better way of generating a final json file

---

- Started putting stuff on AWS
- Began with front-end
	- Changed how `exercises.json` was imported so that it was included in the final build
	- Uploaded `dist/` to s3 bucket
	- Went into Properties and enabled static website hosting
	- S3 needs CloudFront to support https
	- Apparently Amplify just does all this for you and automatically builds from github
		- Reddit says don't use it: [Avoid AWS Amplify at all cost - it's fraud](https://www.reddit.com/r/nextjs/comments/1685dyr/avoid_aws_amplify_at_all_cost_its_fraud/)
	- Had to made sure `index.html` was at the root of S3 bucket
	- Woop woop! Got the front end working on S3 and CloudFront!
	- CloudFront caches responses from S3 for 24 hours so when you rollout a new version of the web app it might now show for a while
		- Can just invalidate the CloudFront cache
		- Invalidate all files with `/*`
	- So I've basically got to manually upload new files to the S3 bucket every time
		- Think I can automate this: [Automate static website deployment from Github to S3 using AWS CodePipeline](https://medium.com/avmconsulting-blog/automate-static-website-deployment-from-github-to-s3-using-aws-codepipeline-16acca25ebc1)

+ Next, back-end server
	+ Installed java 21, built a jar file, uploaded it to the server and ran it
	+ Connection refused when connecting to EC2 public IP and a port number
	+ Tried making server accept CORS from any address
	+ Tried pinging the server from within (SSH) using localhost - worked
	+ Tried pinging the EC2 public IP - worked
	+ Tried creating a python http server on EC2 and couldn't connect to that with public ip and port number - so it's not spring boot...
	+ Not really sure what the problem was in the end, just remade the instance, with a better understanding of security groups and ACLs, and now it's working
+ Finally, the database
	+ It was already in amazon RDS but when I tried to send a request to the server to create an account it wouldn't work
	+ Had to edit it's security group to allow traffic from the server's security group (I had previously set it up to only allow traffic from my home IP address)

- Server wasn't running when I closed all terminal windows on my machine
- Apparently can use `nohup` or `tmux` to keep something running after closing a terminal, but not the best for long term server running
- Better to use `systemd`
	- Will sort that out later

+ Front end connects to the server with the right url in `.env.production.local`
	+ CORS being annoying and stopping the connection working properly
+ Enabled CORS for the domain of the S3 bucket but the EC2 instance is accessed through HTTP whilst the S3 bucket is accessed through HTTPS so it gets upset
+ Need to enable HTTPS for the spring boot server

---

Enabled HTTPS for Spring Boot

SSL password - gymapp
CN=bob higgins, OU=boghiggins, O=bobhiggins, L=bobhiggins, ST=bobhiggins, C=bh

can't get the front end to connect to the back end

- https
	- can't get the back end to use https since that requires an ssl certificate from a certificate authority which requires a domain which costs money
- http
	- can't get it to work if the front-end uses http because of cross-site cookies which require https (been the case with chrome 80 back in 2020)
