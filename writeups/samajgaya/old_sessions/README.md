# Old Sessions - Web Exploitation (easy)

The challenge description hints at some sort of session token that doesn't expire.
Getting into the page, I register and create a new user for myself.
Then login with this new user.

![screenshot of home-page](home-page.png "Home Page")

Right of the bat, I notice two things:
1. `Admin` user, probably what I want to get into
2. A comment by `mary_jones_8992`
    >  Hey I found a strange page at /sessions 

Navigating to /sessions:

![screenshot of /sessions](sessions.png "/sessions")

Notice the session token of `admin` is just right there? Simply entering this
sesssion token

![screenshot of firefox session](input-session.png "input the token")

Navigate back to the home-page, and we have our flag!
