---
title: "Dockerizing a Nodejs Express App"
date: 2022-08-02T07:50:07+05:00
draft: false
---

Dockerizing a node express app is simple. 

## step1: Create simple node app with express.
1. package.json with with its dependencies.
2. index.js with required code which will be creating and running server at port 8080.

Our simple server code that returns hello world when we will hit http://x.x.x.x:8080/

```json
//package.json
{
  "name": "node-express-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.1"
  }
}


```


```javascript
//index.js
'use strict';

const express = require('express');

// Constants
const PORT = 8080;
const HOST = '0.0.0.0';

// App
const app = express();
app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);
```

 

### step2: Write docker file named `Dockerfile`
1. Specify node docker image
2. Assume working directory where all our code will reside inside image.
3. Copy code that working directory.
4. install npm packages
5. expose port 8080
6. run server command


```bash
mkdir node-express-app
cd node-express-app
npm init -y
npm install express --save
```

```docker
FROM node:16

# Create app directory
WORKDIR /usr/src/app


# Bundle app source
COPY . .
RUN npm install


EXPOSE 8080
CMD [ "node", "index.js" ]

```

### Step3: Create .dockerignore file
you will need dockerignore file just like .gitignore where you have to exclude directories like node_modules etc

```txt
node_modules
npm-debug.log

```

### Step4: Build your image
you have to run build command where you have created Dockerfile. i.e root of of your project directory.
the build command will read your Dockerfile and will perform things in series. 
while running build command you have to specify your image name also or the tag that will identify your image.



```bash
#Run build command
docker build . -t node-express-app
#list docker images
docker images

```

### Step5: Run container from your image
The live form of your image is called container. While running container from image we will specify port in the form of `EXTERNAL_PORT:INTERNAL_PORT`
There is `-d` flag is used to specify run container in background rather than interactive. you can say here d stand stand for daemon or something that runs in backgorund.

```bash
#docker run -p EXTERNAL:INTERNAL --detach <image_name>
docker run -p 44444:8080 -d node-web-app

#list running container
docker ps

#get logs from container
docker logs -f <container_id>

#Entering and executing into running container
docker exec -it <container_id> /bin/bash
```

### Step6: Testing your node app
you server is running at port 8080 while you have externall mapped it to 44444 so we can make a curl request on that url.
```bash
curl -i localhost:44444

#stop container
docker stop <container_name>

#remove the stopped container
docker rm container_name

```



