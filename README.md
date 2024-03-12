# Tensorflow Convolutional Neural Network Testing on User MNIST Digits
(c) Syd Strzempko (stzuko.github.io)
Updated 3/12/2024, originally written 2018

## Deployment

Access the app at https://tfjs-on-mnist.onrender.com

## Local Deployment Steps, 2024 Notes

__Note__: As of Mar 2024,  several packages on original project are outdated and require older versions of node/npm or require substantial reworking in code if updated.
I used a few workarounds in order to minimize impact on original project, as well as support on Windows for node-gyp requires additional steps and installs to function. Outlined below are the troubleshooting steps I needed to take to establish  new instance on Render.

### node & npm version management (with nvm):

Current versions: node - 18.16.1, npm - 9.5.1

Several stackoverflow posts led me to believe I would need an older version of node/npm in order to allow `node-gyp` to build, and I experienced several issues with bindings (related to @tensorflow packages). Prior, I believed I needed v9.11.2/v5.6.0 in conjunction, pulled from [history of node/npm releases](https://nodejs.org/en/about/previous-releases) around initial project creation.

In order to ensure that packages that would need reworking with updates do not pull latest version with npm caret versioning syntax, `package.json` was modified to specify exact versions of the dependencies used in initial project.

### Microsoft Build Tools (Windows-specific)

In order to compile tensorflow C binaries, Windows systems need Windows Build Tools
Given the difficulties experienced with `node-gyp`, a specific version is needed to be installed and then specified in this project like this:

```
npm install --global --production windows-build-tools --vs2015
npm config set msvs_version 2015 –global
```

It is also important to have Python 2.7 installed on your system and have that version be used by node. To specify, can set version with either Windows PATH variables or with 

```
npm config set python python2.7
```

* Note, this `config set python` command will not register python as a valid keyword on newer versions of npm, I have noticed

References for this step:
[Article](https://spin.atomicobject.com/node-gyp-windows/)
[Original StackOverflow post](https://stackoverflow.com/questions/50286109/how-to-get-node-gyp-working-on-windows-10)

### Canvas

In dependencies, [Canvas](https://www.npmjs.com/package/canvas) was originally used to train my tensorflow model, and the output of that trained model was saved in the `/model/` folder. The code used in training is still present in `server.js` file but commented out, as once the model was trained, in my RESTful app the model was referenced to predict new inputs rather than being trained each time a user would send a new request. In attempting to get a new instance of this project established, I came across difficulties with outdated Canvas package and thus had to comment out version from `package.json`:

```
"canvas": "2.1.0",
```

as well as write mock functions for those imported by Canvas in order to maintain readability of code in `server.js`. However, this means that any attempt to generate a new model from dataset or retrain model would require reinstalling this package and dealing with the issues mentioned above.

### Other Updates

Updated client to register a proxy for `/model` endpoint with `http-proxy-middleware` rather than relying on `package.json` configuration to do so.

Reference:
[Article](https://bobbyhadz.com/blog/react-could-not-proxy-request-to-localhost)

Also needed to modify model file imports using `@tensorflow/tfjs-node` package preload functionality rather than direct filenames

Reference:
[StackOverflow](https://stackoverflow.com/a/53766926)


## Libraries Used

* [Tensorflow JS](https://js.tensorflow.org/)
* [Tensorflow for Nodejs](https://github.com/tensorflow/tfjs-node)
* [MNIST images](cs.nyu.edu/~roweis/data.html)
* [Create-react-app](https://github.com/facebook/create-react-app)
* [Canvas](https://www.npmjs.com/package/canvas)
* [React-chartkick](https://github.com/ankane/react-chartkick)

## Description

* Model is pretrained on MNIST data; see `server.js` for associated image manipulation - data is shuffled by iterating over each digit set, sorted into batches and trained

* Model is saved with `save` functionality into `/model` on Express server side

* Upon user input, canvas data gets passed to server as a POST request

* The server loads the saved model and weights, converts the request data to tensors, then runs `predict` on the tensors

* We are using an accuracy metric across 10 output values (coded to each digit class); data returned reflects our network's shape

## Code Referenced

* [Tensorflow.js official tutorial](https://js.tensorflow.org/tutorials/mnist.html)

* [Unofficial tutorial](https://gogul09.github.io/software/digit-recognizer-tf-js)

* [Express with Create-React-App tutorial](https://medium.freecodecamp.org/how-to-make-create-react-app-work-with-a-node-backend-api-7c5c48acb1b0)

* [Drawable Canvas](https://enlight.nyc/projects/web-paint/)
