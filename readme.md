mkdir your-app-name
cd your-app-name


## 1. **Initialize Python environment**

```Shell
$ python2 -m venv venv
$ . venv/bin/activate
```

## 2. **Install basic requirements for base program**

```Shell
$ pip install flask
$ pip freeze > requirements.txt
```

## 3. **Write the basic program**
Make your first python file called `app.py`.

```Python
from flask import Flask

app =Flask(__name__)

@app.route("/")
def index:
    return "Hello World!"

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

- The basic app is now functional and requests the browser to return the text 'Hello World!' to us. 

## 4. **Version Control is a Must for Web Deployment**
### Local Git Repo Initialization
- `git init`
- add a .gitignore file to ignore __pycache__ and venv.

```Shell
$ echo venv > .gitignore
$ echo __pycache__ >> .gitignore
$ git add .gitignore app.py requirements.txt
$ git commit -m "Initialize Git repository"
```

## Working with Remotes.
- `git commit -m "Initial commit"`
- `git remote add origin http://github.com/username/ypur-app-name.git `
- run `git remote -v` to check.
- Force rename the main brnach as "main": `git branch -M main`
- if all is okay and as expected, `git push -u origin <main>`.

## 5. **Preparing for Deployment**
### First Steps with Heroku
- Sign up to Heroku for the first time (if not previously completed).
- Install Heroku CLI app and make sure its added to path.
- `heroku login -i`
- for your first time, try `heroku login` from the web UI and save that API key for future reference. it'll be the password for the CLI app instaed of your actual password.

### Intial Config for Deployment
Setup the config locally.

```Shell
$ echo "web:gunicorn app:app" > Procfile
$ pip install gunicorn
$ pip freeze > requirements.txt
```

Make it a habit to save your progress ...

```Shell
$ git add Procfile requirements.txt
$ git commit -m "Add Heroku deployment file"

## 6. **Create the App**

```Shell
$ heroku create <some-unqiue-app-name>
$ git heroku push main
```

After pushing the master branch to the heroku remote, you’ll see that the output displays ormation about the building and deployment process. Your app URL is somewhere in there.
t this point the app is live, grab that URL and look it up in your browser to view.

## 7. **Make your first changes.**
Make a small change to the app.py file:

```Python
from flask import Flask

app =Flask(__name__)

@app.route("/")
def index:
    return "Hello this is the new version!"

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

We've just changed the return text.
Make it a habit to save your progress ...

```Shell
$ git add app.py
$ git commit -m "Change the welcome message"
$ git push heroku main
```

With these commands, you commit the changes to the local Git repository and push them to the heroku remote. This triggers the building and deployment process again. You can repeat these steps whenever you need to deploy a new version of your application. You’ll notice that subsequent deployments usually take less time because the requirements are already installed.

## 8. Establishing the Deployment Workflow.
- Implement a workflow for your application deployment using Heroku pipelines. This particular workflow uses three separate environments called local, staging, and production. 
- This kind of setup is widely used in professional projects since it allows testing and reviewing new versions before deploying them to production and putting them in front of real users.
    - Development is the local environment.
    - Staging is the preproduction environment used for previews and testing.
    - Production is the live site accessed by final users.
- Adding a staging environment can greatly benefit the development process. The main purpose of this environment is to integrate changes from all new branches and to run the integration tests against the build, which will become the next release.

- Let's get started:

```Shell
$ heroku create your-app-name-staging --remote staging
$ git push staging main
$ heroku pipelines:create -a your-app-name
$ ? Pipeline name your-app-name-pipeline
$ ? Stage of your-app-name production
$ heroku pipelines:add your-app-name-pipeline -a your-app-name-staging
$ ? Stage of your-app-name-staging staging
```

- The pipeline now consists of 2 apps: your-app-name and your-app-name-staging.
- For more information or a more complex pipeline construction, please refer to the official Heroku documentation.

## 9. Using the Deployment Workflow
As an example of the deployment workflow in action, let's add a minor edit to the app.py file again:

```Python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return "This is yet another version!"
```

You can deploy this new version to your staging environment by running the following commands:

```Shell
$ git add app.py
$ git commit -m "Another change to the welcome message"
$ git push staging master
```

These commands commit app.py and push the changes to the staging remote, triggering the building and deployment process for this environment. You should see the new version deployed at https://realpython-example-app-staging.herokuapp.com/.

Note that the production environment is still using the previous version.

When you’re happy with the changes, you can promote the new version to production using the Heroku CLI: `heroku pipelines:promote --remote staging`.

## 10. An Important Side Note on Costing.
Oh! Btw... these apps run at a cost (no free lunch, baby). So, to save costs, if you want to turn them off, you could run the command:`heroku ps:scale web=0 -a your-app-name`

Note: You need to specify which app here. This eseentially turns off the dyno which supports your web hosting (which as of the time of this writing, is $0.01 per hr). Remember to re-assign the dyno when you want to revive your app.

## Managing Settings and Secrets for Different Environments
Most applications require different settings for each environment to do things like enabling debugging features or pointing to other databases. Some of these settings, like authentication credentials, database passwords, and API keys, are very sensitive, so you must avoid hard-coding them into the application files.

You can create a config.py file to hold the non-sensitive configuration values and read the sensitive ones from environment variables. In the following code block, you can see the source code for config.py:

```Python
import os

class Config:
    DEBUG = False
    DEVELOPMENT = False
    SECRET_KEY = os.getenv("SECRET_KEY", "this-is-the-default-key")

class ProductionConfig(Config):
    pass

class StagingConfig(Config):
    DEBUG = True

class DevelopmentConfig(Config):
    DEBUG = True
    DEVELOPMENT = True
```

This code declares a Config class used as the base for each environment’s configuration. Note that on line 6, SECRET_KEY is read from an environment variable using os.getenv(). This avoids disclosing the actual key in the source code. At the same time, you can customize any option for each environment.

Next, you have to modify app.py to use a different configuration class depending on the environment. This is the full source code of app.py:

```Python
import os
from flask import Flask

app = Flask(__name__)
env_config = os.getenv("APP_SETTINGS", "config.DevelopmentConfig")
app.config.from_object(env_config)

@app.route("/")
def index():
    secret_key = app.config.get("SECRET_KEY")
    return f"The configured secret key is {secret_key}."
```

The configuration is loaded from one of the previously defined classes in config.py. The specific configuration class will depend on the value stored in the APP_SETTINGS environment variable. If the variable is undefined, the configuration will fall back to DevelopmentConfig by default.

Note: For this example, the message was modified to show the SECRET_KEY obtained by app.config.get(). You don’t typically display sensitive information as part of your responses. This is just an example to show how you can read these values.

Now you can see how this works locally by passing some environment variables when launching the app:

```Shell
$ SECRET_KEY=key-read-from-env-var flask run
```

The above command sets the SECRET_KEY environment variable and starts the application. If you navigate to http://localhost:5000, then you should see the message The configured secret key is key-read-from-env-var.

Next, commit the changes and push them to the staging environment by running the following commands:

```Shell
$ git add app.py config.py
$ git commit -m "Add config support"
$ git push staging master
```

These commands commit changes in app.py and the new config.py file to the local Git repository and then push them to the staging environment, which triggers a new building and deployment process. Before proceeding, you can customize the environment variables for this environment using the Heroku CLI:

```Shell
$ heroku config:set --remote staging \
  SECRET_KEY=the-staging-key \
  APP_SETTINGS=config.StagingConfig
```

Using the config:set command, you’ve set the value of SECRET_KEY and APP_SETTINGS for staging. You can verify that the changes were deployed by going to https://realpython-example-app-staging.herokuapp.com/ and checking that the page shows the message The configured secret key is the-staging-key.

Using Heroku CLI, you can also get the values of the environment variables for any app. The following command gets all the environment variables set for the staging environment from Heroku:

```Shell
$ heroku config --remote staging
=== realpython-example-app-staging Config Vars
APP_SETTINGS: config.StagingConfig
SECRET_KEY:   the-staging-key
```

As you can see, these values match the previously set ones.

Finally, you can promote the new version to production with different configuration values using the Heroku CLI:

```Shell
$ heroku config:set --remote prod \
  SECRET_KEY=the-production-key \
  APP_SETTINGS=config.ProductionConfig
$ heroku pipelines:promote --remote staging
```

The first command sets the values of SECRET_KEY and APP_SETTINGS for the production environment. The second command promotes the new app version, the one that has the config.py file. Again, you can verify that the changes were deployed by going to https://realpython-example-app.herokuapp.com/ and checking that the page shows The configured secret key is the-production-key.

You learned how to use a different configuration for each environment and how to handle sensitive settings using environment variables. Remember that in real-world applications, you shouldn’t expose sensitive information like SECRET_KEY.