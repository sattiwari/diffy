# Diffy

[![Build status](https://img.shields.io/travis/twitter/diffy/master.svg)](https://travis-ci.org/twitter/diffy)
[![Coverage status](https://img.shields.io/codecov/c/github/twitter/diffy/master.svg)](https://codecov.io/github/twitter/diffy)
[![Project status](https://img.shields.io/badge/status-active-brightgreen.svg)](#status)
[![Maven Central](https://img.shields.io/maven-central/v/com.twitter/diffy_2.11.svg)](https://maven-badges.herokuapp.com/maven-central/com.twitter/diffy_2.11)

## Status

This project is used in production at Twitter and is being actively developed and maintained.

## What is Diffy?

Diffy finds potential bugs in your service using running instances of your new code and your old
code side by side. Diffy behaves as a proxy and multicasts whatever requests it receives to each of
the running instances. It then compares the responses, and reports any regressions that may surface
from those comparisons. The premise for Diffy is that if two implementations of the service return
“similar” responses for a sufficiently large and diverse set of requests, then the two
implementations can be treated as equivalent and the newer implementation is regression-free.

## How does Diffy work?

Diffy acts as a proxy that accepts requests drawn from any source that you provide and multicasts
each of those requests to three different service instances:

1. A candidate instance running your new code
2. A primary instance running your last known-good code
3. A secondary instance running the same known-good code as the primary instance

As Diffy receives a request, it is multicast and sent to your candidate, primary, and secondary
instances. When those services send responses back, Diffy compares those responses and looks for two
things:

1. Raw differences observed between the candidate and primary instances.
2. Non-deterministic noise observed between the primary and secondary instances. Since both of these
   instances are running known-good code, you should expect responses to be in agreement. If not,
   your service may have non-deterministic behavior, which is to be expected.

Diffy measures how often primary and secondary disagree with each other vs. how often primary and
candidate disagree with each other. If these measurements are roughly the same, then Diffy
determines that there is nothing wrong and that the error can be ignored.

## How to get started?

Here’s how you can start using Diffy to compare three instances of your service:

1. Deploy your old code to `localhost:9990`. This is your primary.
2. Deploy your old code to `localhost:9991`. This is your secondary.
3. Deploy your new code to `localhost:9992`. This is your candidate.
4. Download the latest Diffy binary or build your own from the code.
5. Run the Diffy jar with following command line arguments:

    ```
    java -jar diffy-server.jar \
    -candidate=\"localhost:9992\" \
    -master.primary=\"localhost:9990\" \
    -master.secondary=\"localhost:9991\" \
    -service.protocol=\"http\" \
    -serviceName=\"ConversationService\" \
    -proxy.port=:31900 \
    -admin.port=:31159 \
    -http.port=:31149
    ```

6. Send a few test requests to your Diffy instance:

    ```
    curl localhost:31900/your_application_route
    ```

7. Watch the differences show up in your browser at [http://localhost:31149](http://localhost:31149).