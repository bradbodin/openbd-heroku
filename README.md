OpenBD on Heroku

Dependencies:

1. A Working JVM
2. Maven 3 (is built into OSX Lion, on other platforms you will likely have to install it)
3. Foreman (if you want to be able to use the Procfile locally - nice but not required)
4. Herok Toolbelt

To get it running locally:

1. Check out this repo.
2. Run "mvn package"
3. Run "foreman start" (or copy the comannd out of the Procfile to run the jetty-runner)

To deploy to Heroku:

1. Run "heroku create --stack cedar myAppName"
2. Run "git remote heroku add"
3. Run "git push heroku master"
4. Run "heroku open"