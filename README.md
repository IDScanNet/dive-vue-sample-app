# Project setup
1. Install project dependencies
```
npm install
```

2. Copy over the neural network binary files to the public directory
```
npm run copyNeuralNetworks
```

# Building and Running the Application
1. Compile and run the application locally
```
npm run serve
```

2. Compile and minify the application for production
```
npm run build
```



# Deploy to Firebase

1. Install the Firebase CLI

```
npm install -g firebase-tools
```

2. Login in to Firebase
```
firebase login
```

3. Setup the firebase project we are deploying to
```
firebase init
```
- Select `Yes` you are ready to proceed
- Select only the `Hosting` option
- Create a new project in your firebase account for this deployment
- Use `dist` as your public directory
- Select `Yes` when asked if you want to configure as a single-page app
- Select `No` when asked if you want to set up automatic builds and deployments with GitHub
- Select `No` when asked if you want to overwrite the `dist/index.html` file


4. Deploy the project to firebase

```
firebase deploy
```

