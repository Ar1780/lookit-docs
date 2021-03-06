Setup for local development
===================================

These instructions will walk you through setting up to run Lookit locally.

Overview
--------

Even though we may just be changing the frame definitions in
ember-lookit-frameplayer, we will need to install *both* the the Django app
(``lookit-api``) and the Ember app (``ember-lookit-frameplayer``), tell
them how to talk to each other, and run both of those servers locally.

- On Lookit, we will add some basic information to our superuser, and
then add a child and demographic data. 
- We then create a study locally.
- In ember-lookit-frameplayer, we'll add a token which gets added to the headers of the API requests so that Lookit knows about the logged-in user making the request. 
- We can then navigate directly to the study from the Ember app to bypass the build process locally.

This will enable you to make changes to frames locally and immediately see
the results of those changes, participating in a study just as if you
were a participant on the Lookit website.

Django App steps
----------------

1. Follow the instructions to install the `django
   app <install-django-project.html>`__ locally. Run the server.

2. Navigate to http://localhost:8000/__CTRL__/ to log in to Lookit. Use the superuser
   credentials created in the django installation steps.

3. Once you are in the Admin app, navigate to users, and then select
   your superuser. If you just created your django app, there should be
   two users to pick from, your superuser, and an anonymous user. In
   that case, your superuser information is here
   http://localhost:8000/__CTRL__/accounts/user/2/change/.

4. Update your superuser information through the admin app. Fill out the
   bold fields:

   -  Family Name: *Your last name*
   -  Identicon: *If no identicon, just type random text here*
   -  Timezone: *America/New_York, as an example*
   -  Locale: *en_US, as an example*
   -  Place a check in the checkbox by “Is Researcher”

   Click “Save”.

5. Create a token to allow the Ember app to access the API by navigating
   to http://localhost:8000/__CTRL__/authtoken/token/. Click “Add Token”,
   find your superuser in the dropdown, and then click “Save”. You will
   need this token later.

6. Create a study by navigating to
   http://localhost:8000/exp/studies/create/. Fill out all the fields.
   The most important field is the ``structure``, where you define the
   frames and the sequence of the frames. Be sure the frame and the
   details for the frame you are testing are listed in the structure.

7. Add demographic information to your superuser (just for testing
   purposes), so your superuser can participate in studies. Navigate to
   http://localhost:8000/account/demographics/. Scroll down to the
   bottom and hit “Save”. You’re not required to answer any questions,
   but hitting save will save a blank demographic data version for your
   superuser.

8. Create a child by navigating to
   http://localhost:8000/account/children/, and clicking “Add Child”.
   Fill out all the information with test data and click “Add child”.

Now we have a superuser with attached
demographic data and a child. We’ve created a study, as well as a token
for accessing the API. Leave the django server running and switch to a
new tab in your console.

   Remember: The OAuth authentication used for access to Experimenter
   does not work when running locally. You can access Experimenter by
   first logging in as your superuser, or by giving another local user
   researcher permissions using the Admin app.

Ember App steps
---------------

1. Follow the instructions to install the `ember
   app <ember-app-installation.html>`__ locally.

2. If you
   make changes to the frames, you should see notifications that files
   have changed in the console where your ember server is running, like
   this:

   ::

      file changed components/exp-video-config/template.hbs

3. Add your token and lookit-api local host address 
   to the ember-lookit-frameplayer/.env file. This will allow your Ember app to talk
   to your local API. Your .env file will now look like this:

   ::

      PIPE_ACCOUNT_HASH='<account hash here>'
      PIPE_ENVIRONMENT=<environment here>
      LOOKIT_API_KEY='Token <token here>'
      LOOKIT_API_HOST='http://localhost:8000'

4. If you want to use the HTML5 video recorder, you’ll need to set up to
   use https locally. Open ``ember-lookit-frameplayer/.ember-cli`` and
   make sure it includes ``ssl: true``:

   .. code:: js

      "disableAnalytics": false,
      "ssl": true

   Create ``server.key`` and ``server.crt`` files in the root
   ``ember-lookit-frameplayer`` directory as follows:

   ::

      openssl genrsa -des3 -passout pass:x -out server.pass.key 2048
      openssl rsa -passin pass:x -in server.pass.key -out server.key
      rm server.pass.key
      openssl req -new -key server.key -out server.csr
      openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt

   Leave the challenge password blank and enter ``localhost`` as the
   Common Name.

5. Run the ember server: ``ember serve``

Starting up once initial setup is completed
-------------------------------------------

This is much quicker! Once you have gotten through the initial setup
steps, you don’t need to go through them every time you want to work on
something.

1. Start the Django app:

   ::

      $ cd lookit-api
      $ source VENVNAME/bin/activate
      $ python manage.py runserver
      

2. Start the Ember app:

   ::

      $ cd ember-lookit-frameplayer
      $ ember serve

3. Log in as your local superuser at http://localhost:8000/__CTRL__/

Previewing or participating in a study
---------------------------------------

To participate in a study locally, you need demographic data and a child
attached to the logged in user, as well as a study. To fetch studies, navigate to
http://localhost:8000/api/v1/studies/. Copy the id of the study you
created earlier. To fetch children, navigate to
http://localhost:8000/api/v1/children/. Copy the id of your child.

Both previewing and participating will save data to your local server; there's no difference in the experience. Preview responses simply have an "is_preview" field set to True, and are displayed differently on the consent manager and individual responses views.

To preview a study, you need to either have read permissions for the study or the study needs to have "shared preview" set to true. To participate, you do not need any particular permissions. 

To participate in a study, navigate to
https://localhost:4200/studies/study_id/child_id, replacing study_id and
child_id with the ids you obtained from the API. (For simplicity,
bookmark this link while you’re working!)

To preview, you can instead navigate to https://localhost:4200/exp/studies/study_id/child_id/preview/, replacing study_id and
child_id with the ids you obtained from the API.


Where does my video go?
-----------------------

If you have set up the Pipe recorder environment variables as described
in `the installation instructions <ember-app-installation.html>`__,
video recorded during your local testing will go to Pipe and then to an
S3 bucket for Lookit development video. Please get in touch if you need access to this video. Depending on the project you are working on, we may provide credentials for accessing the dev S3 bucket, or may ask that you set up your own free Pipe account and have it forward data to you own S3 bucket, which will allow you to test more of the process. (In this case you will use ngrok to send a Pipe webhook to your own local instance.)


Further Reading / Useful Links
--------------------------------

- http://emberjs.com/
- http://ember-cli.com/
- Development Browser Extensions
  - https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi
  - https://addons.mozilla.org/en-US/firefox/addon/ember-inspector/
