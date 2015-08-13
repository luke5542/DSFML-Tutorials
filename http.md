Web requests with HTTP
=====

Introduction
---

DSFML provides a simple HTTP client class which you can use to communicate with HTTP servers. "Simple" means that it supports the most basic features of HTTP: POST, GET and HEAD request types, accessing HTTP header fields, and reading/writing the pages body.

If you need more advanced features, such as secured HTTP (HTTPS) for example, you're better off using a true HTTP library, like libcurl.

For basic interaction between your program and an HTTP server, however, it should be enough.

Http
---

To communicate with an HTTP server you must use the [Http](http://dsfml.com/dsfml/network/http.html) class.

```D
import dsfml.network;

Http http = new Http();
http.setHost("http://www.some-server.org/");

// or
Http http = new Http("http://www.some-server.org/");
```

Note that setting the host doesn't trigger any connection. A temporary connection is created for each request.

The only other function in [Http](http://dsfml.com/dsfml/network/http.html), sends requests. This is basically all that the class does.

```D
Http.Request request = new Http.Request();
// fill the request...
Http.Response response = http.sendRequest(request);
```

Requests
---

An HTTP request, represented by the [Http.Request](http://dsfml.com/dsfml/network/http.html) class, contains the following information:

+ The method: POST (send content), GET (retrieve a resource), HEAD (retrieve a resource header, without its body)
+ The URI: the address of the resource (page, image, ...) to get/post, relative to the root directory
+ The HTTP version (it is 1.0 by default but you can choose a different version if you use specific features)
+ The header: a set of fields with key and value
+ The body of the page (used only with the POST method)

```D
Http.Request request = new Http.Request();
request.setMethod(Http.Request.Method.Post);
request.setUri("/page.html");
request.setHttpVersion(1, 1); // HTTP 1.1
request.setField("From", "me");
request.setField("Content-Type", "application/x-www-form-urlencoded");
request.setBody("para1=value1&param2=value2");

Http.Response response = http.sendRequest(request);
```

DSFML automatically fills mandatory header fields, such as "Host", "Content-Length", etc. You can send your requests without worrying about them. DSFML will do its best to make sure they are valid.

Responses
---

If the [Http](http://dsfml.com/dsfml/network/http.html) class could successfully connect to the host and send the request, a response is sent back and returned to the user, encapsulated in an instance of the [Http.Response](http://dsfml.com/dsfml/network/http.html) class. Responses contain the following members:

+ A status code which precisely indicates how the server processed the request (OK, redirected, not found, etc.)
+ The HTTP version of the server
+ The header: a set of fields with key and value
+ The body of the response

```D
Http.Response response = http.sendRequest(request);
writeln("status: ", response.getStatus());
writeln("HTTP version: ", response.getMajorHttpVersion(), ".", response.getMinorHttpVersion());
writeln("Content-Type header:", response.getField("Content-Type"));
writeln("body: ", response.getBody());
```

The status code can be used to check whether the request was successfully processed or not: codes 2xx represent success, codes 3xx represent a redirection, codes 4xx represent client errors, codes 5xx represent server errors, and codes 10xx represent DSFML specific errors which are not part of the HTTP standard.

Example: sending scores to an online server
---

Here is a short example that demonstrates how to perform a simple task: Sending a score to an online database.

```D
import dsfml.network;
import std.conv;

void sendScore(int score, const std.string& name)
{
    // prepare the request
    Http.Request request = new Request("/send-score.php", Http.Request.Post);

    // encode the parameters in the request body
    string body = "name=" ~ to!string(name) ~ "&score=" ~ to!string(score);
    request.setBody(body);

    // send the request
    Http http = new Http("http://www.myserver.com/");
    Http.Response response = http.sendRequest(request);

    // check the status
    if (response.getStatus() == Http.Response.Ok)
    {
        // check the contents of the response
        writeln(response.getBody());
    }
    else
    {
        writeln("request failed");
    }
}
```

Of course, this is a very simple way to handle online scores. There's no protection: Anybody could easily send a false score. A more robust approach would probably involve an extra parameter, like a hash code that ensures that the request was sent by the program. That is beyond the scope of this tutorial.

And finally, here is a very simple example of what the PHP page on server might look like.

```D
<?php
    $name = $_POST['name'];
    $score = $_POST['score'];

    if (write_to_database($name, $score)) // this is not a PHP tutorial :)
    {
        echo "name and score added!";
    }
    else
    {
        echo "failed to write name and score to database...";
    }
?>
```
