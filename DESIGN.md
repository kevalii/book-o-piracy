# Design

## Contents
1. [Preliminaries](#preliminaries)
2. [`app.py`](#apppy)
	1. [Config](#config)
	2. [DB Setup](#db-setup)
	3. [Flask Routes](#flask-routes)
3. [`tools/messaging.py`](#toolsmessagingpy)
	1. [Sendgrid](#sendgrid)
	2. [Messaging Functions](#messaging-functions)
3. [`tools/translate.py`](#toolstranslatepy)
4. [`tools/doc.py`](#toolsdocpy)
	1. [Doc Functions](#doc-functions)
5. [`static/editor.js`](#staticeditorjs)

## DESIGN for CS50
### Preliminaries

In my project proposal, I wrote that users would be able to login and have access to a personal history of messages. I ultimately decided against including this feature because while it would've been rather simple to implement, it was an unnecessary obstruction to the ideal user story. Why would someone take the trouble of signing up to use what is essentially a joke site? And if most users will end up using the site only once, then does it really make sense to implement a login feature and personalized history?

Instead of implementing that, I dedicated my efforts to creating an dynamic preview feature to help users in formatting their messages, which I think is far more relevant to the user story. 

### `app.py`

#### Config
`app.config['UPLOAD_FOLDER']` is the path to folder in the app root where user-uploaded files are stored. This is admittedly a somewhat dangerous practice, but we apply several checks to our parsing of the files to minimize most vulnerabilities. 

#### DB Setup
`heroku = Heroku(app)` enables Heroku to set env variables and connect to the PostgreSQL database.

`db = SQLAlchemy(app)` initializes SQLAlchemy

`class Message` is the schema through all successfully translated messages will be committed to the database. 

#### Flask Routes
`index` is largely self-explanatory. However, I will note here that throughout the app we `session.pop('message', None)` to delete any message cookies. 

`translate POST` gets all the form data and validates an uploaded file. `check_file(file.filename)` ensures that the file has the correct extension. `secure_filename(file.filename)` formats the filename in a such way to nullify certain malicious inputs. The data of the message is then stored as a cookie in `session['message']`.

`preview GET` gets the message data from `session['message']` and displays it on the page.

`preview POST` does the same, sends the message, and commits the data to the database.

In both `preview GET` and `preview POST` we include a try-except clause to ensure that cookie has been created. Otherwise, the app will fail to retrieve the cookie at times.

### `tools/messaging.py`

#### SendGrid
I initially wrote the messaging component using Python's `smtp` and `email` libraries (as in `tools/old_messaging.py`, but when I pushed the project to Heroku, I discovered that my implementation of it would not consistently work during production. [Heroku even recognizes this!](https://help.heroku.com/CFG547YP/why-am-i-getting-errors-when-sending-email-with-gmail-via-smtp) So instead I took their advice and used one of Heroku's many messaging addons (SendGrid in this case). 

#### Messaging Functions
`send_message` sends the data from `compose_message` and sends an HTTP request to SendGrid's API with a JSON containing the email headers and content. 

`compose_message` initializes an mail object that becomes a JSON object with all the relevant headers to be included in a request to SendGrid's API when `mail.get()` is called. `body` contains the primary message content. `personalization` is used add email addresses to `mail.to`. This function also handles adding an attachment to the email if applicable. Before adding the attachment, we encode the file in base64 (this is an SMTP standard; otherwise the attachment will be unreadable) and set up the appropriate HTTP headers.

### `tools/translate.py`
It queries the translation API and returns the translated message. 

### `tools/doc.py`

#### Doc Functions
`get_docx` opens a `.docx` file, translates the paragraph within (nearly all text in the body is considered a paragraph), and saves the file.

`get_text` opens a `.txt` file, translates the text within, and saves the file.

`escape` replaces any newlines and replaces it with `<br>`. The translation API refuses to return a translation when a request contains newlines (probably to nullify malicious inputs), so we must ecsape the newlines out of a message before sending it off. This is not really necessary in `get_docx` but is crucial to `get_text` and translating the main message in `app.py`.

### `static/editor.js`

This handles the editor and the dynamic preview features.

#### Editor Functions
`updatePreview` gets the text from the textarea (id `translation_text`) containing the user-inputted message and displays it as HTML in the `format-preview` div. This function is called in `getActiveText` after the addition of HTML tags to reflect the formatting of the updated text and is also called whenever the document initializes (for displaying formatted translated text in `preview.html`).

`getActiveText` gets the selected text in the textarea and prepends/appends the appropriate HTMl tags according to the value in `arg`. This function is called whenever a button is clicked in `translate.py`.

