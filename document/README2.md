# TP 2

Q 2-1: Testcontainers is a Java library that lets you run Docker containers during your tests.

when we run mvn clean verify, Testcontainers automatically starts a real PostgreSQL container. Our integration tests then connect to this temporary database. This is very useful because it checks if our code works with a real database, not just a "mock" or fake one.

_____________________________________________________________________________________________________

Q 2-2:  we use secrets to protect sensitive data like passwords or tokens. If we write our password directly in the .yml file, everyone who can see our code can steal it. GitHub Secrets are encrypted and are only given to the action when it runs, keeping our information safe.

_____________________________________________________________________________________________________

Q 2-3: this creates a dependency. It means the "build and push images" job will only start if the test-backend job finished successfully. We don't want to publish new Docker images if our tests are failing. It's a key part of CI/CD.

_____________________________________________________________________________________________________

Q 2-4: we push images to a central registry (like Docker Hub) to share them. This allows other developers on our team to use them, or (more importantly) it allows our production servers to pull the new image and deploy it.


