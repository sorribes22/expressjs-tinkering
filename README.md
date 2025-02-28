# Express tinkering project
Project to do the first steps with Express and NodeJS.
It has been developed following a [midudev tutorial](https://youtube.com/playlist?list=PLUofhDIg_38qm2oPOV-IRTTEKyrVBBaU7).

## Steps followed to create this project
### Create a docker container
If you don't have any language interpreter installed in your computer, like me, you can use [Docker](https://docs.docker.com/engine/install/) to package, run and deploy your project.
Create a simple Dockerfile from the official NodeJS image (please, use a [Node.js LTS version](https://nodejs.org/en/about/previous-releases#nodejs-releases)).
```dockerfile
FROM node:18
WORKDIR /usr/src/app
USER node
COPY --chown=node:node . .
EXPOSE 3000
```

May be you will need some service in the future running in your project. Create a `docker-compose.yml` file to manage the services. You will need [Docker compose](https://docs.docker.com/compose/install/) installed too.
```yaml
version: '3.8'
services:
  app:
    container_name: express-tinkering
    build:
      context: .
    volumes:
      - .:/usr/src/app
    command: tail -f /dev/null
```
> Take in consideration that the `volumes` property is to mount the current directory in the container. This way you can edit the files in your computer and the changes will be reflected in the container. This is useful for development, but you should not use it in production.
> The command in `command` section is to keep the container running, so we can access it and prepare the project.

At this step we are now ready to set up the project.
Run docker-compose up -d to build (if it's the first time) and start the container.
```bash
docker-compose up -d
```

If you want to check the container health, you can use the following command:
```bash
docker ps
```
Use `docker logs express-tinkering` to check the container logs if any problem.

Now we can access the container and init the project.
```bash
docker exec -it express-tinkering bash
```

### Init a NodeJS project
Once we have the container running we can set up our express project.
Run `npm init` inside the container to create the `package.json` file. The `-y` flag is to skip the questions and use the default values.
After that, we can install the standard package to enforce a code style.
- The `-D` flag is to save the package as a development dependency. You will not need it in production.
- And the `-E` flag is to save the package as an exact version.

```bash
npm init -y
npm install standard -DE
```
Edit the `package.json` file to add the [eslint](https://eslint.org/) config:

```json
{
  "dependencies": {
    ...
  },
  "eslintConfig": {
    "extends": "standard"
  }
}
```

You will also need to configure standard in your IDE.
If you are using [Visual Studio Code](https://code.visualstudio.com/), you can install the [ESLint plugin](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint).

Ensure your vscode setting json file is configured as follows:
```json
{
  ...,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "[javascript]": {
    "editor.formatOnSave": true
  }
}
```

### Set up our first HTTP server
Now we are going to install the express package and create the first server.
```bash
npm install express -E
```

Stop the container by running `docker-compose down` and edit the `Dockerfile` like this to install the dependencies. Since we have the `package.json` file we can install them every time we build the container.

```dockerfile
FROM node:18
WORKDIR /usr/src/app
USER node
COPY --chown=node:node . .
RUN npm install
EXPOSE 3000
```

Create a `src/index.js` file with the following content:
```javascript
const express = require('express')
const app = express()
const PORT = process.env.PORT || 3000

app.get('/', (req, res) => {
  res.send('Hello World')
})

app.use((req, res) => {
  res.status(404).send('Not Found')
})

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`)
})
```

Modify the `package.json` file to add the `dev` script to run the server in watch mode. This way the server will restart every time we change the code.
Change the main definition too.
```json
{
  "name": "express-tinkering",
  "version": "1.0.0",
  "main": "src/index.js",
  "scripts": {
    "dev": "node src/index.js --watch"
  },
  ...
}
```

Edit the `docker-compose.yml` file to map the port 3000 of the container to the port 3000 of the host machine. Modify the `command` property to run the server.
```yaml
version: '3.8'
services:
  app:
    container_name: express-tinkering
    build:
      context: .
    volumes:
      - .:/usr/src/app
    ports:
      - 3000:3000
    command: npm run dev
```

Start the container again with `docker-compose up -d` and access the server in your browser at `http://localhost:3000`.

### Adding another layer
#### Remove `x-powered-to` header
Add the next line in `src/index.js` to prevent our api clients to know which technology and version is running the server.
```javascript
app.disable('x-powered-by')
```

#### Configure the server to use JSONs
If you want to use only JSONs to struct your input/output data ensure your api server knows it.
```javascript
app.use(express.json())
```

## References
- [Good NodeJS Docker practises](https://softwarecrafters.io/devops/docker-nodejs-buenas-practicas)
- [StandardJS](https://standardjs.com/)
- [HTTP codes explained by cats](https://http.cat/)
- [HTTP codes explained by engineers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)