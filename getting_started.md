## Getting started
*Before starting your journey with Sealious, make sure [Node.js](https://nodejs.org/en/) is installed and set. Also we'll be using [MongoDB](https://www.mongodb.org/) in our examples.*

1. Open command line in your desired directory.
2. Type in `npm init`. This is create `package.json` file, which will hold all crucial data about your project.
    * You will be prompted several times, you can just `ENTER` through.
3. Once `package.json` is set, type in `npm install --save sealious`. This will download Sealious from `npm` (Node Package Manager) and save the dependency in your `package.json`.
4. We need to have some means of communication with the server. Type in `npm install --save sealious-www-server` and `npm install --save sealious-channel-rest`.
5. To set the database, type in `npm install --save sealious-datastore-mongo`.

And that's it. Note that Sealious reads its dependencies from `package.json`, so don't forget to create it.