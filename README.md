# Book O' Piracy
## for CS50

### Preliminaries
Book O' Piracy is a website built on Flask and run on Gunicorn that at its core uses a translation API to convert user-inputted text in "pirate speak" and send to a specified email address(es). The production website can be found at [Book O' Piracy](https://book-o-piracy.herokuapp.com/). As the url might suggest, the app is hosted on Heroku and deployed using Heroku Git, although the project has a separate [GitHub repo](https://github.com/kevalii/book-o-piracy) that is updated alongside the website. 

Because the app and its components were written in mind with deployment on Heroku, running the app locally requires some setup, and the messaging (and to a lesser extent the database) functionality of the app will be lost since it inherently relies on integration with certain Heroku addons (SendGrid and Heroku Postgresql).

I'm unfortunately unable to share the contents of the Postgres database, because
...a) the database is stored on Heroku's servers and cannot be locally stored, which was primarily motivated by how
...b) storing the database within application root itself is just generally bad practice.

### Testing Setup
1. Install [PostgreSQL](https://www.postgresql.org/), make sure PostgreSQL is online, and initialize a database named 'translations' in psql bash session by entering `psql` and then `createdb translations`
	1. Then in the same psql session enter `\q` to quit, and then, in the regular bash session, enter `psql translations`
	2. Enter `CREATE TABLE messages(id SERIAL, translation TEXT NOT NULL, time TIMESTAMP NOT NULL);` 
	3. Check that the table exists via `\d`
	4. Enter `\q` to quit once more
Then either:
2. Clone a repo containing the source code as modified in below by running `git clone https://github.com/kevalii/piracytest.git` in bash while in some directory.

OR, if you'd prefer to see the exact changes below,

2. Clone the primary repo and obtain the source code by running `git clone https://github.com/kevalii/book-o-piracy.git` in bash while in some directory.
3. In the root directory of the cloned project, install the dependencies described in requirements.txt via `pip install -r requirements.txt` (You may want to setup a virtualenv for this step).
3. Then
	1. Uncomment line 22 of `app.py` such that it looks like
	```python
	app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://localhost/translations'
	```
	2. Comment line 31 of `app.py` such that it looks like
	```python
	# heroku = Heroku(app)
	```
	3. Modify line 115 of `app.py` such that it looks like
	```python 			
	text = text
	```
	4. Modify line 8 of `tools/doc.py` such that it looks like
	```python
	para.text = para.text
	```
	5. Modify line 19 of `tools/doc.py` such that it looks like
	```python
	file.write(text)
	```

These changes will enable you to run the site on localhost, but without the translation component (since it requires an API key) nor the messaging component.

If you'd like to test the site with a roughly equivalent messaging component, you could do the following:

1. Modify line 5 of `app.py` so that it looks like
```python
from tools.old_messaging import compose_message, send_message
```
2. Modify line 145 of `app.py` so that it looks like
```python
msg = compose_message(message['addressee'], 'parcel from ' + message['addresser'], text, path.join(app.config['UPLOAD_FOLDER'], message['file']) if message['file'] is not None else None)
```
and append to it this line
```
send_message(msg)
```

This legacy messaging tool goes into setting up an SMTP connection that should make it possible to send messages when hosting the app on localhost. This way, you don't need to go through SendGrid! Be warned that messages sent using the legacy tool will likely end up in your spam folders. 

If you *really, really* need to test the translation component for some reason, just email me at alexrankine@college.harvard.edu, and I'll give you the API key such that you'll replace line 6 of `tools/translate.py` to look like

```python
headers = {'X-FunTranslations-Api-Secret': API-KEY-HERE}
```

The above, excluding the translation API key, should be enough to run the website locally and test most everything except for the translation component.

There is a config file for the application exclusively on the Heroku Git repo that is solely used to retrieve different API keys and credentials when the app is hosted on Heroku, so I'm omitting it from this source code. 