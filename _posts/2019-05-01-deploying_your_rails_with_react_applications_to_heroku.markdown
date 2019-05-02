---
layout: post
title:      "Deploying your Rails with React Applications to Heroku"
date:       2019-05-01 21:13:40 -0400
permalink:  deploying_your_rails_with_react_applications_to_heroku
---


So, you’ve been programming and developing all these fantastic apps for a few months now, you’ve put lots of hours and effort, and you’re extremely proud of them. You’ve shown them to friends and family when you have your computer available to do so. But then a friend asks you for a link or how to find and use the app online. With a blank stare, you say they can only be used from your computer at this point. Bummer! 

The next logical step after spending time developing your applications, and once you're satisfied with your work, is to deploy them. In essence, to move out of the development environment and into the production environment. This isn’t a straight forward task and usually requires some careful tweaking to make sure everything runs smoothly on the web.

For this article, in an attempt to save you some headaches and countless trial and error commits and erroneous deploys, I will go in-depth on deploying a Rails with React.js application using Heroku.

For this article, I’m assuming a few things:

1.    You have a working Rails application with React.js as the client on the same stack that can run on your local environment
2.    You used the  `rails new –api`  command to build the app in API mode (best practice in this case)
3.    You used `create-react-app` to build the React client
4.    Your Ruby version is 2.4.0 or above 
5.    Your Rails version is 5.x
6.    You know what Heroku is and have set up an account and have their CLI installed
7.    You have Yarn or NPM installed (we’ll use Yarn in this article)


Before you can even attempt deploying the app, you must first make some changes to the default configurations in Rails and the overall project in general.

I suggest you create a separate git branch for these coming changes in case you want to go back to your original development version. Once your deployment/production version is complete, you can merge to your master branch.

Ok, let’s get to it!


**Changing the Rails Database**

By default, Rails uses SQLite3 for its database, however, SQLite is not intended as a production-grade database. Heroku works with PostgreSQL, a more robust database. If you’re not familiar with PostgreSQL, I encourage you to research it; It’s really powerful and used by many applications.

To move from sqlite3 to postgres, open your Gemfile and remove or comment out the sqlite3 gem and add the postgres gem:

`# gem ‘sqlite3’
gem ‘pg’
`

Alternatively, you can group your gems on their respective environments if you want to preserve your current database on the development environment:

```
group :development do
    gem ‘sqlite3’
end

group :production do
    gem ‘pg’
end
```

At this point, if you don’t have one, I suggest you create a seed file based on your current database. This will allow you to move or recreate your current data into the new PostgreSQL database. If you plan on having a blank production database, then this won’t be necessary.

Once your gemfile has been updated, re-install the dependencies by running  `bundle install` in your terminal.

By adding the postgres gem, we’ve told Rails to start using PostgreSQL as the database. For this to work, you need to change the database adapter from `sqlite3` to `postgresql` in the database configuration file `config/database.yml`:

```
development:
  adapter: postgresql
  database: my_database_development
  pool: 5
  timeout: 5000
test:
  adapter: postgresql
  database: my_database_test
  pool: 5
  timeout: 5000

production:
  adapter: postgresql
  database: my_database_production
  pool: 5
  timeout: 5000
```


*Note: This setup makes all environments use PostgreSQL. If you want to keep using SQLite on development, leave the development adapter pointing to sqlite3. Also, make sure you substitute “mydatabase” to your app’s root directory name.*

At this point, you should commit your changes to your production branch.


**ActiveStorage Changes**

Rails’ ActiveStorage allows you to attach files to ActiveRecord models. If you are using ActiveStorage to attach pictures, audio, or other files, you need to make some configuration changes before deployment. This is because Heroku uses an “ephemeral” disk that allows you to write files to it, however, those files won’t persist after the Heroku app is restarted. 

Instead of storing uploaded files directly on Heroku, it is recommended to use a cloud file storage service such as AWS or Google Cloud. In this article, we’ll use Google Cloud Storage as it’s very straight forward and it’s free to use for the first year.

Got to https://cloud.google.com and sign in with your Google account or create one. You’ll be taken to the Google Cloud Platform where you will create a new project. Name the project as your application’s name:

![]file:///Users/ale/Desktop/ScreenShots/Screen%20Shot%202019-04-30%20at%2021.04.18.png
 
Next, open the Google Cloud Platform menu on the left side. Under “Storage”, hover over “Storage” and click on “Browser”. Click on the “Create Bucket” button to create a new “bucket” for your project:

 

A bucket is where your data is held on the Google Cloud. The name must be globally unique across Cloud Storage. For the settings, select Regional storage class, choose your location, and create:

 

Your bucket is now created. Now you have to give it access to your Rails backend. To do so you need to get an authorization key. On the main menu, under API’s & Services”, click on “Credentials”. This will open up a new page to create new credentials; create a new Service account key:

  
Pick a name for your service account, select Project Owner as the Role, and make sure the Key Type is in JSON format:

 
After clicking “Create”, a public/private key pair is generated and download to your computer. Make sure to keep this in a safe place as it is the only copy of it you will have.

You now have a working Google Cloud project to upload and host your application’s files. Now let’s set up the Rails backend to work with it:

You need to let your app know about the authorization keys you just saved. First create a `secrets` folder inside your app’s `config` folder.

Add `config/secrets/*` to the app’s `.gitignore`file. This will prevent the contents of the `secrets` folder to be included in git version control, and uploaded and shared to git remotes.

Now rename the downloaded .json file to: `your-app-name.json` and move it into the  `config/secretes` folder.

You now need to configure Rails to use the Google Cloud Service. First, add the Google Storage gem to your gemfile:

`gem "google-cloud-storage", "~> 1.8", require: false`

And run `bundle install` in the terminal.

Next, change the app’s storage configuration file `config/storage.yml`. Make the following changes:
```
google:
     service: GCS
     project: your-app-name
     credentials: <%= ENV['GOOGLE_APPLICATION_CREDENTIALS'] %>
     bucket: your-app-name
```

*Note: Make sure you add your corresponding project and bucket names instead of “your-app-name”.*

Now, tell Rails to use `:google` for the production environment. In `confing/environments/production.rb`, make the following change: 

`config.active_storage.service = :google`

*Note: Your development environment will continue to use ActiveStorage. If you want to use Google Cloud for development environment as well, you need configure Rails to do so:*

```
google_dev:
     service: GCS
     project: your-app-name
     credentials: <%= Rails.root.join("config/secrets/ your-app-name.json ") %>
     bucket: your-app-name
```

`config/environments/development.rb` :

`config.active_storage.service = :google_dev`

At this point, if this was a single Rails application, you could create the database and do your migrations, and then deploy it to Heroku. But since we have a React.js client, we need to change a few more things on the backend. This would be a good time to commit your changes.


**Building the Production Client and Setting Up Rails for Deployment**

Just as with a single React.js application, the client needs to be built to create a `build` directory that will hold the app’s production static files. Included in these files will be` index.html`, which will be served to visitors of the site. These files will be contained on the `public` folder of the Rails backend, which is what we want to deploy to Heroku.
We can use Yarn (or NPM) to create the build and deploy scripts that will prepare our app for production. Heroku works very good with Node.js applications and will recognize such apps if it sees a `package.json` file in the root of the application. Let’s create that file:

In your terminal and in the project’s root directory, execute `yarn init` and follow instructions.

This will add the `package.json` file to the Rails backend that Heroku will then use to build and deploy the app. It will look something like this:

```
{
  "name": "my-awesome-package",
  "version": "1.0.0",
  "description": "The best package you will ever find.",
  "main": "index.js",
  "repository": {
    "url": "https://github.com/yarnpkg/example-yarn-package",
    "type": "git"
  },
  "author": "Yarn Contributor",
  "license": "MIT"
}
```

We now need to add the scripts that will be executed for such actions:

```
"scripts": {
    "build": "cd client && yarn install && yarn build && cd ..",
    "deploy": "cp -a client/build/. public/",
    "postinstall": "yarn build && yarn deploy && echo 'Client built!'"
  },
```

When we push our repo to Heroku, Heroku will run `npm install` while deploying the app to make sure all necessary dependencies are installed. With the scripts above, we are telling Heroku to run the `postinstall` script after it initially runs `npm install`. The `postinstall` script runs the `build` script which will run `yarn install` in the client directory to install any dependencies the client may have, and then run `yarn build` to create the ` build` folder that will hold the static files. Then the `deploy` script will run and will copy all the `build` files into the `public` folder for deployment.

With the above instructions, Heroku now knows how to build and deploy the app. We now need to tell it how to run the Rails server once the app is hosted on Heroku. To do this, we need to give it instructions via a Procfile:

In the app’s root directory, create a file called `Procfil` (it must be named exactly this way). In the file, write the following line:

 `web: bundle exec puma -t 5:5 -p ${PORT:-3001} -e ${RACK_ENV:-development}`

This tells Heroku to run the Rails server on port 3001.

*Note: You can set this port to whatever you want. Just make sure it is not the same port your client is using to connect, and that is the same port your fetch/ajax request are using to fetch data from the backend. More on this in the section below.*


**Connecting the React Client to the Rails API**

By default, the Rails server runs on port 3000. So does the client development build when ruining in your local server, i.e. when running `yarn start` or `npm start`.

You might have been running two local servers by executing `rails s –p 3001` in your terminal and on your apps root directory, and `yarn start` on the client’s root directory. This will allow your client to run on the http://localhost:3000/ and the Rails backend/server on http://localhost:3001/. 

This manual setup is ok while on development, but for production, we need to make sure that the client makes the server calls to the right port, every time.  To do so, we must add a proxy to the client’s `package.json` file:
 
```
{
  "name": "client",
  "version": "0.1.0",
  "private": true,
  "proxy": "http://localhost:3001",
  "dependencies": { 
	
	. . .
	
	}
 
```		


Next, we need to make changes to the client’s fetch requests. 

You may have a component that makes calls to the Rails API/server. Something similar to this:

```
const apiURL = 'http://localhost:3001'

export const fetchWorkouts = () => {
  let data = {
    method: 'GET',
    headers: {
      'Accept': 'application/json',
      'Content-type': 'application/json'
    }
  }

  return dispatch => {
    fetch(`${apiURL}/resources`, data)
      .then(response => response.json())
      .then(resources => dispatch({
          type: 'FETCH_RESOURCES',
          resources
      }))
      .catch(errors => errors)
  }
}
```

A few things to note here. First, this is a React/Redux “API call action”. It uses ‘Redux-Thunk’ to handle the fetch request, but the same principles apply to use other tools like ‘Axios’ or vanilla fetch requests.

Second, note that here we have an `apiURL` variable that points to the Rails server. The variable is then interpolated into the fetch request address where it’s used to call on the “resources” endpoint. Although this is the correct way to handle these calls on your development environment, this wouldn’t work for a production build when deployed to Heroku. This is because we are hardcoding the path to our server and Heroku –after deployment and once the app has been built on their server, uses its own HTTP routing stack with unique paths to the app’s Heroku server.

To fix this, you simply remove the `apiUrl` variable from the fetch request path and leave your resource endpoint. Something along:

 ```
//const apiURL = 'http://localhost:3001'

export const fetchWorkouts = () => {
  let data = {
    method: 'GET',
    headers: {
      'Accept': 'application/json',
      'Content-type': 'application/json'
    }
  }

  return dispatch => {
    fetch(`/resources`, data)
      .then(response => response.json())
      .then(resources => dispatch({
          type: 'FETCH_RESOURCES',
          resources
      }))
      .catch(errors => errors)
  }
}
```

One last thing to consider, Is the use of ‘React-Router’.

If your React client uses ‘react-router’ for its routing, you would need to make some changes to the Rails backend. Specifically, you will need to create a ‘fallback route’ for all routes or paths that don’t match or are not included in your client’s routes. In other words, you will tell your app to serve the client’s `index.html` for any path that is not included in your client’s routes.

To do so, in the Rails < app/application_controller.rb> file, add the following:

```
class ApplicationController < ActionController::API

  def fallback_index_html
    render :file => 'public/index.html'
  end
  
end
```

And in the Rails `congif/routes.rb`  file, add the following: 

```
get '*path', to: "application#fallback_index_html", constraints: ->(request) do
    !request.xhr? && request.format.html?
  end
```

All right, that was a lot to do! But now our application is ready to be deployed. 
Next, we’ll create the Heroku app and deploy our Rails with React app to it.

This will be a good time to commit your work.


**Deploying to Heroku**

As a reminder, I’m assuming you have a Heroku account and have the Heroku CLI installed.
If you haven’t, please go to https://www.heroku.com/, create an account and follow instructions on how to set up the CLI.

The first thing to do is to log in to your Heroku account. In your terminal, execute `heroku login` and fill in your credentials.

Now, create a new Heroku app:

In your terminal, and in your app’s root directory, execute the following:
`heroku create your-app-name`

You can confirm the new Heroku app is connected to your app by executing `git remote –v`.
This will list all your remote depositories associated with your app directory. You should see Heroku listed there because Heroku uses your git repo to build your app on their server.

Next, you will tell Heroku how to build the app. Since the app uses Node.js and Ruby on the stack, Heroku needs to execute the correct buildpacks when building the app. To do so, execute the following in your terminal:

`heroku buildpacks:add heroku/nodejs --index 1`
`heroku buildpacks:add heroku/ruby --index 2`

You are all set!!

Commit all your changes and push them to Heroku:


`git add .`
`git commit -m "first Heroku deploy "`
`git push heroku master`


*Note: the < git push > command takes in two arguments, a remote name and the local branch name to be pushed, i.e ` git push <remotename> <branchname`. You should push the branch name where all these changes took place. If you followed my suggestion to create a new deployment branch before making these changes, you will need to deploy that branch instead of master above.*

*When you push your app to Heroku, Heroku will run the scripts we set in the Rails `package.json` file to build the app. This process will take a minute or two. You can see the `build` and `deploy` scripts running in the background while this process takes place.*

And finally!... since this is a brand new app hosted in Heroku, there won’t be a database set yet. You must create it, migrate it, and seed it (if you have a seed file). In your terminal:

`rake db:create`
`rake db:migrate`
`rake db:seed`

**And that’s it! You should now have a working production app hosted in Heroku.**

You can open and test the app by running `heroku open` on the root directory.

One final note: if you find that your app’s images or styling aren’t loading properly, it may be because their corresponding static assets aren’t being processed while deploying. Check the Heroku logs to see which assets aren’t loading. 

The quick fix is to pre-compile your assets before deployment. Note, however, that doing the following will slow down the app’s loading time. It’s recommended instead that you host the static assets on a CDN service.

To precompile assets, change the following lines in the Rails < config/environments/production.rb > file:

`config.assets.complie = true`
`config.serve_static_assests = true`

If you are new to deployment, I hope this guide has saved you a few hours of research and have successfully deployed your app. If you are experienced, hopefully, you can use this guide as a refresher and maybe keep it as a comprehensive reference for the future. I sure will.

**References:**

https://devcenter.heroku.com/articles/getting-started-with-rails5

https://medium.com/@bruno_boehm/reactjs-ruby-on-rails-api-heroku-app-2645c93f0814

https://www.youtube.com/watch?v=Z0sFlwz0YvU

https://medium.com/@pjbelo/setting-up-rails-5-2-active-storage-using-google-cloud-storage-and-heroku-23df91e830f8


 












