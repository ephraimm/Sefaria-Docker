# Moderator Tools

 ( the instructions in https://developers.sefaria.org/docs/local-installation-instructions#run-with-docker-compose  need some changes to configuration to get this up and running locally, I'd recommend rather cloning:

 

In order to access moderatortools `http://localhost:8000/modtools` while running a local instance or via docker, you need to setup a Django staff account.

## Register a Staff User

### Step 1: Setup Google Storage Account
Sefaria stores gravatars and other media in a Google Cloud Storage Account, as part of the user registration process user profile pic is stored in this storage account and needs to be setup before proceeding, the credentials can be created and downloaded by following the instructions found at: `https://developers.google.com/workspace/guides/create-credentials#service-account`

The downloaded file should be saved in `web/config/<google_service_account_credentials>.json`

which will be accessed via the docker web instance via the local settings and config path mounted in docker-compose:

    //docker-compose.yml
    ...
        volumes:
            - ./web/local_settings.py:/www/sefaria/local_settings.py
            - ./web/config:/www/sefaria/config
    ...


the credentials are accessed in  `local_settings.py
` file

    //local_settings.py
    GOOGLE_APPLICATION_CREDENTIALS_FILEPATH = "sefaria/config/<google_service_account_credentials>.json"

### Step 2: Register a user
From the front end navigate to new user registration `http://localhost:8000/register`

### Step 3: Admin permissions
the regular registered user needs raised permissions


python manage.py shell
In the python console, run the following:

    from django.contrib.auth.models import User

Email address is used as the username, enter inside the quotes the email you entered when you registered the user created through the ui (http://localhost:8000/register)


    user = User.objects.get(email='<your email address>') 
    
change the password, if needed, and set the admin staff and super user flags.

    user.set_password('<a secure password>') # 

    user.is_superuser = True
    user.is_admin = True
    user.is_staff = True
    user.save()

exit the Django manage session:

    exit()

### Step 4: Staff Login
When navigation to `http://localhost:8000/modtools` you should be asked to login now use the email adress/password combo you just setup as the staff user to access the Moderator tools and follow the rest of the instructions found on: https://developers.sefaria.org/docs/workflowy#step-2-uploading-the-index-via-moderator-tools


# Note: Connect to SQL Lite

By default Sefaria uses Django to store user credentials, from the web docker container you can create an interactive SQLite session with the details found in the `local_settings.py` config file:

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3', # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
            'NAME': '/db/db.sqlite', # Or path to database file if using sqlite3.
            'USER': '',                      # Not used with sqlite3.
            'PASSWORD': '',                  # Not used with sqlite3.
            'HOST': '',                      # Set to empty string for localhost. Not used with sqlite3.
            'PORT': '',                      # Set to empty string for default. Not used with sqlite3.
        }
    }


using the sql lite file path found in the `NAME` attrib of default database config:

    sqlite3 /db/db.sqlite

once a session is started, query the registered user as follows:

    select * from auth_user;