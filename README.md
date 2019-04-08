# Insight DevOps Engineering Systems Puzzle
## Thoughts progress log

I debugged the system with a top-down approach. 

As a first step towards understanding the whole system and possibly narrowing down the bugs, I used the command `"docker-compose up -d"` to bring up the system and then navigated to http://localhost:8080 in Chrome. Not surprisingly, the browser returned the error message `"url cannot be reached"`. I suspect this to be a config issue, so I looked into the config file "docker-compose.yml" and found the port config `"80:8080"` to be suspicious. After checking the docker documentation on container-networking [1], I confirmed that the right way to config port for this system should be `"8080:80"` instead of the given `"80:8080"`. 

After fixing this issue I retried the whole system. This time I got a new error `"502 Bad Gateway"`. By checking related background info [2] of this error, as well as looking into the system log of the flaskapp service (via `"docker-compose logs flaskapp"`), I realized that the default port of flask app is 5000 while in Dockerfile only the port 5001 is exposed. By looking into some flask tutorials [3], I fixed the issue by changing `app.run(host='0.0.0.0')` to `app.run(host='0.0.0.0', port=5001)` in the app.py file.

After fixing this issue I retried the system again and were happy to see the index page successfully rendered. However, when I filled in the required fields in the form and clicked the "Enter Item" button, I received the output of empty strings separated by comma, which doesn't reveal any useful information. As this time the system didn't crash or send out any error code, I suspect the problem to be with the logic of handling the button click. By looking into the main functional code (i.e., app.py), I found the root cause of this issue, which is to directly return the string representation of a collection `"qry.all()"`. I solved the problem by adding code to iterate through the collection returned by "qry.all()" and concatenate the items within the collection to a human-readable string.

Finally, after all these fixes, I managed to get the system running smoothly and achieve the functionalities as expected.

[1] https://docs.docker.com/config/containers/container-networking/

[2] https://blog.hubspot.com/marketing/502-bad-gateway

[3] http://flask.pocoo.org/docs/dev/quickstart/
