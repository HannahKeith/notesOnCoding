# CONNECT YOUR APP TO HEROKU 
Using: Express, Node, MongoDB, MongoDB Atlas, Heroku

## Make Your Repo
[Create a new repo on Github and clone to your local repo](https://git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository), make sure to include a [.gitignore](https://git-scm.com/docs/gitignore) file that will hide your .env file. If you don't hide your .env file and it's pushed to github, people can steal your password. 

After you're repo is set up, double check that your gitignore file exists and is hiding the .env file
```
#dotenv environment variables file
.env
.env.test
```
Run 'git remote -v' to check you have an origin fetch and push url that matches the repo you made.

If that all goes well, move on to getting your npm all set up
```
npm init
npm install ejs express mongoose method-override dotenv 
//list anything else for authentication, etc.
```
Go to your package.json file in your code editor and make sure "engines" matches your current node version. You can check your current node version by typing "node -v" in the terminal.
```
//package.json file
"engines": {
  "node" : "node version here" 
}
```

##Figure out MongoDB Atlas: 
[MongoDB Start Up Docs](https://docs.atlas.mongodb.com/getting-started/)

> MongoDB Atlas is a fully-managed cloud database developed by the same people that build MongoDB. Atlas handles all the complexity of deploying, managing, and healing your deployments on the cloud service provider of your choice (AWS, Azure, and GCP). - MongoDB Atlas Docs


### After your MongoDB account and free cluster are created, create a user. This is how you get a password. 
* On the atlas website's sidebar, click on "Database Access" under the "Security" section. 
* Click on [ + Add New Database ] button
* Choose password as your authentication method and type in the username and password. Write down the password!
* Set user privileges to "Read and write to any database"

### Make Sure Your IP Whitelist is correct
This sets which IP addresses will be able to access your database. 
* On the atlas website's sidebar, click on "Network Access" under the "Security" section. 
* Click on [ + Add IP Address] button
* If you haven't already allowed all access, you'll see an [+ Allow Access From Anywhere] button. Click on it. If it's not there and you see a table on your Network Access page that lists the IP Address 0.0.0.0/0, you're good to go. 

### Now, get the connection string for your cluster. This will go in your .env file and on your Heroku. 
* View your cluster by clicking on "clusters" on the atlas website's sidebar under the "Data Storage" section. 
* In your cluster, click the "connect" button to access the page that will let you connect the cluster to your code/heroku app.
  * If using Node.js : choose driver version Node.js on the connect page. 
  * Copy your connection string and replace password with the password you created for your user. 

### Back to your local repo... You should have the beginnings of a CRUD app going and an .env file created within the directory you're working in. If you don't have a .gitignore file set up, do it now!
You'll set up an .env file with your port and the connection string from MongoDB Atlas. In your connection string, replace 'password' and the arrows around it <> with your user access password.

```
PORT=3000
MONGODB_URI=mongodb+srv://user:<password>@...
```

Make sure your server.js file is using the port and url from your .env file

```
const PORT = process.env.PORT || 3333; <- If your console records you are connected to 3333, that means you're not linking to the .env file
const MONGODB_URI = process.env.MONGODB_URI;
```

### Now, make sure you have Heroku downloaded and connect Heroku to your local repo using the terminal

[Heroku download site](https://devcenter.heroku.com/articles/heroku-cli#download-and-install) 

After you've downloaded heroku, double check that the install worked in your terminal
```
heroku --version
//you should see the heroku version in the output
```
Login to Heroku, then create your app using a unique name. If you're successful, you'll see a fetch and push path for heroku as well when you run 'git remote -v'

```
heroku login
//Takes you to Heroku site to login or pop-up login within your terminal

heroku create <yournamehere>
//Heroku will tell you if you were successful or if the name is already taken

git remote -v
//should see 4 paths total - 2 origin and 2 heroku
```

### Connect Everything Together
Now, go back to your account on [Heroku](https://dashboard.heroku.com/apps). You should see a dashboard with your newly created app listed. Set up your settings to transfer that MongoDB connection string to your deployed app on Heroku, since you can't take your .env file with you to the internet.
* Click on the app and, in the nav bar on the app show page, click "Settings" 
* Right under the app info section, there will be the section "Config Vars." Click the button [ + Reveal Config Vars] to display the key:value pairs for your app.
  * This is like your replacement .env file for the deployed version of your app. 
* Set your key as "MONGODB_URI" and your value as the configuration string that you set in the .env file within your local repo.

Go back to your terminal. You can now push your code to heroku! The push to heroku will take a minute.

```
git push heroku master
```
You can throw in some tester code in your server.js file to render. When you type 'heroku open' in your terminal, it should load that tester code in your browser window. 

And you're ready to make your app. When you commit and push changes, make sure to push to both 'origin master' and 'heroku master'