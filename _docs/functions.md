---
layout: docs
title: Functions
short_title: Functions
---

# What is a function?

A function is a piece of code that takes one or more pieces of input
and returns something as output.  For example:

{% include side-by-side.html demo="function" %}

Functions allow you to do a lot of different things in
**docassemble**.  This section explains the standard **docassemble**
functions.  If you know how to write [Python] code, you can write your
own functions and include them in your interview using a [`modules`]
block.

# How to use functions

To use the functions described in this section in your interviews, you
need to include them from the [`docassemble.base.util`] module by
writing the following somewhere in your interview:

{% highlight yaml %}
---
modules:
  - docassemble.base.util
---
{% endhighlight %}

Unless otherwise instructed, you can assume that all of the functions
discussed in this section are available in interviews when you include
this [`modules`] block.

# <a name="functions"></a>Functions for working with variable values

## <a name="defined"></a>defined()

As explained in [how **docassemble** runs your code], if your code or
templates refer to a variable that is not yet defined, **docassemble**
will stop what it is doing to ask a question or run code in an attempt
to obtain a definition for that variable.

Sometimes, this is not what you want **docassemble** to do.  For
example, you might just want to check to see if a variable has been
defined yet.

The `defined()` function checks to see if a variable has been
defined.  You give it a name of a variable.

{% include side-by-side.html demo="defined" %}

It is essential that you use quotation marks around the name of the
variable.  If you don't, it is as if you are referring to the variable.

## <a name="value"></a>value()

The `value()` function returns the value of a variable, where the name
of the variable is given as a string.

These two code blocks effectively do the exact same thing:

{% highlight yaml %}
---
code: |
  answer = value('meaning_of_life')
---
{% endhighlight %}

{% highlight yaml %}
---
code: |
  answer = meaning_of_life
---
{% endhighlight %}

Note that `value(meaning_of_life)` and `value("meaning_of_life")` are
entirely different.  The first will treat the value of the
`meaning_of_life` variable as a variable name.  So if
`meaning_of_life` is "chocolate," `value(meaning_of_life)` will
attempt to find the value of the variable `chocolate`.

## <a name="need"></a>need()

The `need()` function takes one or more variables as arguments and
causes **docassemble** to ask questions to define each of the
variables if the variables are not already defined.  Note that with
`need()`, you do _not_ put quotation marks around the variable name.

For example, this [`mandatory`] code block expresses [interview logic]
requiring that the user first be shown a splash screen and then be
asked questions necessary to get to the end of the intererview.

{% highlight yaml %}
---
mandatory: true
code: |
  need(user_shown_splash_screen, user_shown_final_screen)
---
{% endhighlight %}

This happens to be 100% equivalent to writing:

{% highlight yaml %}
---
mandatory: true
code: |
  user_shown_splash_screen
  user_shown_final_screen
---
{% endhighlight %}

So the `need()` function does not "do" anything.  However, writing
`need()` functions in your code probably makes your code more readable
because it helps you convey in "natural language" that your interview
"needs" these variables to be defined.

## <a name="force_ask"></a>force_ask()

Usually, **docassemble** only asks a question when it encounters a
variable that is not defined.  However, with the `force_ask` function,
you can cause such a condition to happen manually, even when a
variable is already defined.

In this example, we use `force_ask` to cause **docassemble** to ask a
question that has already been asked.

{% highlight yaml %}
---
modules:
  - docassemble.base.util
---
question: |
  Are you a communist?
yesno: user_is_communist
---
mandatory: true
code: |
  if user_is_communist and user_reconsidering_communism:
    user_reconsidering_communism = False
    force_ask('user_is_communist')
---
question: |
  I suggest you reconsider your answer.
field: user_reconsidering_communism
---
mandatory: true
question: |
  % if user_is_communist:
  I am referring your case to Mr. McCarthy.
  % else:
  I am glad you are a true American.
  % endif
{% endhighlight %}

This may be useful in particular circumstances.  Note, however, that
the effect of `force_ask()` is temporary.  If the user refreshes the
screen while looking at the `user_is_communist` question a second
time, it will be as though `force_ask` never happened.

Note also that no code that comes after `force_ask()` will ever be
executed.  Once the `force_ask()` function is called, the code stops
running, and the question indicated by the variable name will be
shown.  That is why, in the example above, we set
`user_reconsidering_communism` to False before calling `force_ask`.
The variable `user_reconsidering_communism`, which had been set to
`True` by the "I suggest you reconsider your answer" question, is set
to `False` before the call to `force_ask` so that the [`mandatory`]
code block does not get stuck in an infinite loop.

A different way to reask a question is to use the built-in Python
operator `del`.  This makes the variable undefined.  Instead of
writing:

{% include side-by-side.html demo="force-ask" %}

we could have written:

{% include side-by-side.html demo="del" %}

This will also cause the `user_is_communist` question to be asked
again.  This is more robust than using `force_ask` because the user
cannot get past the question simply by refeshing the screen.

The `force_ask()` function can also be given the names of variables
that refer to [`event`] blocks.  The screen will be shown, but no
variable will be defined.

## <a name="force_gather"></a>force_gather()

The `force_gather()` function is similar to [`force_ask()`], except it
is not only asks a question, but insists that a variable be defined.

In addition to doing what [`force_ask()`] does, `force_gather()`
engages the assistance of the [`process_action()`] function to make
sure that the variable is defined.  The [`process_action()`] function
will not finish until the variable is defined.

In order for `force_gather()` to work as intended, you need to include
[`process_action()`] in an [`initial`] <span></span> [`code`] block.

{% include side-by-side.html demo="force-gather" %}

In this example, [`force_ask()`] would not have worked in place of
`force_gather()`, because if the user selects "Something else," the
interview would not continue on to the next question that offers to
define `favorite_food`.  Calling `force_ask('favorite_food')` means
"look for a question that offers to define `favorite_food`, and
present it to the user, whereas calling
`force_gather('favorite_food')` means "until `favorite_food` is
defined, keep asking questions that offer to define `favorite_food`."

Normally, you will not need to use either [`force_ask()`] or
[`force_gather()`]; you can just mention the name of a variable in
your [`question`]s or [`code`] blocks, and **docassemble** will make
sure that the variables get defined.  The [`force_ask()`] and
[`force_gather()`] functions are primarily useful when you are using
[actions] to do things that are outside the normal course of the
[interview logic].

# <a name="special responses"></a>Functions for special responses

## <a name="message"></a>message()

{% include side-by-side.html demo="message" %}

The `message()` function causes **docassemble** to stop what it is
doing and present a screen to the user that contains a given message.

By default, the user will be offered an "exit" button and a "restart"
button, but these choices can be configured.

The first argument is the title of the screen the user will see (the
[`question`]).  The second argument, which is optional, indicates the
text that will follow the title (the [`subquestion`]).

The `message()` function also takes keyword arguments.  The following
do the same thing:

* `message("This is the big part of the question", "This is the
  smaller part of the question")`
* `message(question="This is the big part of the question", subquestion="This is the
  smaller part of the question")`

The optional keyword arguments influence the appearance of the screen:

* `message("Bye!", "See ya later", show_restart=False)` will show the
  Exit button but not the Restart button.
* `message("Bye!", "See ya later", show_exit=False)` will show the
  Restart button but not the Exit button.
* `message("Bye!", "See ya later", url="https://www.google.com")`:
  clicking the Exit button will take the user to Google.
* `message("Bye!", "See ya later", show_leave=True)` will show a
  Leave button instead of the Exit button.
* `message("Bye!", "See ya later", show_leave=True, show_exit=True,
  show_restart=False)` will show a Leave button and an Exit button.
* `message("Bye!", "See ya later",
  buttons=[{"Learn More": "exit", "url": "https://en.wikipedia.org/wiki/Spinning_wheel"}])`
  will show a "Learn More" button that exits to Wikipedia.

## <a name="response"></a>response()

The `response()` command allows the interview author to use code to
send a special HTTP response.  Instead of seeing a new **docassemble**
screen, the user will see raw content as an HTTP response, or be
redirected to another web site.  As soon as **docassemble** runs the
`response()` command, it stops what it is doing and returns the
response.

There are four different types of responses, which you can invoke by
using one of four keyword arguments: `response`, `binaryresponse`,
`file`, and `url`.  There is also an optional keyword argument
`content_type`, which determines the setting of the
[Content-Type header].  (This is not used for `url` responses,
though.)

The four response types are:

* `response`: This is treated as text and encoded to UTF-8.  For
  example, if you have some data in a dictionary `info` and you want
  to return it in [JSON] format, you could do
  `response(response=json.dumps(info),
  content_type='application/json')`.  If the `content_type` keyword
  argument is omitted, the [Content-Type header] defaults to
  `text/plain; charset=utf-8`.
* `binaryresponse`: This is like `response`, except that the data
  provided as the `binaryresponse` is treated as binary bytes rather
  than text, and it is passed directly without any modification.  You
  could use this to transmit images that are created using a software
  library like the [Python Imaging Library].  If the `content_type`
  keyword argument is omitted, the [Content-Type header] defaults to
  `text/plain; charset=utf-8`.
* `file`: The contents of the specified file will be delivered in
  response to the HTTP request.  You can supply one of two types of
  file designators: a [`DAFile`] object (e.g., an assembled document
  or an uploaded file), or a reference to a file in a **docassemble**
  package (e.g., `'moon_stars.jpg'` for a file in the static files
  folder of the current package, or
  `'docassemble.demo:data/static/orange_picture.jpg'` to refer to a
  file in another package).
* `url`: If you provide a URL, the web server will redirect the user's
  browser to the provided URL.

Here is an example that demonstrates `response`:

{% include side-by-side.html demo="response" %}

Here is an example that demonstrates the `binaryresponse`:

{% include side-by-side.html demo="response-svg" %}

The following example shows how you can make information entered into
an interview available to a third-party application through a URL that
returns data in [JSON] format.

{% include side-by-side.html demo="response-json" %}

Note the following about this interview.

1. We load [`docassemble.base.util`] so that the
   [`interview_url_action()`], [`process_action()`], and
   [`url_action()`] functions are available.
2. We set [`multi_user`] to `True` in order to disable server-side
   encrpytion.  This allows an external application to access the
   interview without logging in as the user.
3. The `query_fruit` [`event`] code will be run as an [action] when
   someone accesses the link created by [`interview_url_action()`].
   In order to enable actions to operate in this interview, we call
   [`process_action()`].

The `response()` command can be used to integrate a **docassemble**
interview with another application.  For example, the other
application could call **docassemble** with a URL that includes an
interview file name (argument `i`) along with a number of
[URL arguments].  The interview would process the information passed
through the URLs, but would not ask any questions.  It would instead
return an assembled document using `response()`.

{% highlight yaml %}
---
modules:
  - docassemble.base.util
---
attachment:
  name: A file
  file: test_file
  variable name: the_file
  valid formats:
    - pdf
  content: |
    Hello, world!
---
mandatory: true
code: |
  response(file=the_file.pdf)
---
{% endhighlight %}

Here is a link that runs this interview.  Notice how the name "Fred" is
embedded in the URL.  The result is an immediate PDF document.

> [{{ site.demourl }}?i=docassemble.base:data/questions/examples/immediate_file.yml&name=Fred]({{ site.demourl }}?i=docassemble.base:data/questions/examples/immediate_file.yml&name=Fred){:target="_blank"}

## <a name="command"></a>command()

{% include side-by-side.html demo="exit" %}

The `command()` function allows the interview author to trigger an
exit from the interview using code, rather than having the user click
an `exit`, `restart`, `leave` or `signin` button.

The first argument to `command()` is one of the following:

* `restart`: deletes the user's answers and puts them back at the
  start of the interview.
* `exit`: deletes the user's answers and redirects them to a web site
* `leave`: redirects the user to a web site without deleting the
  user's answers.
* `signin`: redirects the user to the sign-in screen.

The optional keyword argument `url` provides a URL to which the user
should be redirected.

One use of `command()` is to delete interviews after a period of
inactivity.  See [scheduled tasks] for more information.

# Text transformation functions

## <a name="from_b64_json"></a>from_b64_json()

Takes a string as input, converts the string from base-64, then parses
the string as [JSON], and returns the object represented by the
[JSON].

This is an advanced function that is used by software developers to
integrate other systems with docassemble.

## <a name="space_to_underscore"></a>space_to_underscore()

If `user_name` is `John Wilkes Booth`,
`space_to_underscore(user_name)` will return `John_Wilkes_Booth`.
This is useful if you do not want spaces in the filenames of your
[attachments].

## <a name="single_paragraph"></a>single_paragraph()

`single_paragraph(user_supplied_text)` will replace any linebreaks in
`user_supplied_text` with spaces.  This allows you to do things like:

{% highlight yaml %}
---
question: Summary of your answers
subquestion: |
  When I asked you the meaning of life, you said:

  > ${ single_paragraph(meaning_of_life) }

field: ok_to_proceed
---
{% endhighlight %}

If you did not remove line breaks from the text, then if the
`meaning_of_life` contained two consecutive line breaks, only the
first paragraph of the answer would be indented.

{% highlight yaml %}
---
sets: user_done
question: Thanks!
subquestion: Here is your letter.
attachment:
  - name: A letter for ${ user_name }
    filename: Letter_for_${ space_to_underscore(user_name) }
    content file: letter.md
---
{% endhighlight %}

# <a name="actions"></a>Functions for interacting with the interview using URLs

## <a name="url_action"></a><a name="process_action"></a>url_action() and process_action()

The `url_action()` and `process_action()` functions allow users to
interact with **docassemble** using hyperlinks embedded in questions.

`url_action()` returns a URL that, when clicked, will perform an
action within **docassemble**, such as running some code or asking a
question.  Typically the URL will be part of a [Markdown] link inside
of a [question], or in a `note` within a set of [fields].

In order for the action to be performed, you need to create a block of
[`initial`] code containing the command `process_action()`.
The action will be carried out by this function.

Here is an example:

{% highlight yaml %}
---
modules:
  - docassemble.base.util
---
initial: true
code: |
  process_action()
---
mandatory: true
code: |
  if lucky_information_confirmed:
    need(final_screen)
---
field: lucky_information_confirmed
question: |
  Please confirm the following information.
subquestion: |
  Your lucky number is ${ lucky_number }.
  [Increase](${ url_action('set_number_event', increment=1) })
  [Decrease](${ url_action('set_number_event', increment=-1) })

  Also, your lucky color is ${ lucky_color }.
  [Edit](${ url_action('lucky_color') })
---
question: What is your lucky color?
fields:
  - Color: lucky_color
---
sets: final_screen
question: Thank you
subquestion: |
  You have confirmed that your lucky color is ${ lucky_color }
  and your lucky number is ${ lucky_number }.
buttons:
  - Restart: restart
---
event: set_number_event
code: |
  lucky_number += action_argument('increment')
---
code: |
  lucky_number = 2
---
{% endhighlight %}

([Try it out here]({{ site.demourl }}?i=docassemble.demo:data/questions/testaction.yml){:target="_blank"}.)

When the user clicks one of the links, the interview will load as
usual (much as if the user refreshed the web browser page).  The only
difference is that **docassemble** sees the additional information
stored in the URL and makes that information available to the
interview.

In the above example, when the user clicks on the link generated by
`url_action('lucky_color')`, the interview will load and
**docassemble** will process the [`initial`] code block, which runs
the function `process_action()`.  The `process_action` function sees
that there was a URL "action" called 'lucky_color'.  It will look for
a `question` or `code` block that defines `lucky_color`.  (Internally,
it calls [`force_ask()`].)  Since there is a question in the interview
that offers to define `lucky_color`, that question will be asked.

When the user clicks on the link generated by
`url_action('set_number_event', increment=1)`, the `process_action()`
function will look for a `question` or `code` block that defines
`set_number_event`.  It will find the `code` block that was labeled
with `event: set_number_event`.  (See [Setting Variables] for more
information about [`event`]s.)  It will then run that code block.
Note how the [Python] code within that block knows the value of
`increment` which had been specified in the `url_action` function: it
retrieves it with [`action_argument()`].

The `process_action` function runs every time the screen loads, but it
will do nothing if the user did not click on a link generated by
`url_action`.

You can pass as many named parameters as you like to an "action."  For
example:

{% highlight yaml %}
---
question: Hello
subquestion: |
  You can set lots of information by [clicking this link](${
  url_action('set_stuff', fish='trout', berry='strawberry',
  money=65433, actor='Will Smith')}).
---
event: set_stuff
code: |
  info = action_arguments()
  user_favorite_fish = info['fish']
  user_favorite_fruit = info['berry']
  if info['money'] > 300000:
    user_is_rich = True
  actor_to_hire = info['actor']
---
{% endhighlight %}

In this example, we use [`action_arguments()`] to retrieve all of the
arguments to [`url_action()`] as a [Python dictionary].

You can control whether and when the "action" will be performed by
placing the `process_action()` statement in a particular place in your
[`initial`] code.  For example, you might want to ensure that actions
can only be carried out if the user is logged in:

{% highlight yaml %}
---
initial: true
code: |
  if user_logged_in():
    process_action()
---
{% endhighlight %}

Note that these links will only work for the current user, whose
access credentials are stored in a cookie in his or her browser.  It
is possible for actions on the interview to be run by a "third party."
For information on how to do this, see [`interview_url_action()`] and
[scheduled tasks].

## <a name="action_menu_item"></a>action_menu_item()

One way to let the user trigger "[actions]" is to provide a selection in
the menu of the web app.  You can do this by setting the `menu_items`
list.  See [special variables] section for more information about
setting menu items.

{% highlight yaml %}
---
mandatory: true
code: |
  menu_items = [ action_menu_item('Review Answers', 'review_answers') ]
---
{% endhighlight %}

In this example, a menu item labeled "Review Answers" is added, which
when run triggers the action "review_answers."

The `action_menu_item(a, b)` function returns a [Python dictionary] with
keys `label` and `url`, where `label` is set to the value of `a` and
`url` is set to the value of `url_action(b)`.

## <a name="interview_url"></a>interview_url()

The `interview_url()` function returns a URL to the interview that
provides a direct link to the interview and the current variable
store.  This is used in [multi-user interviews] to invite additional
users to participate.

People who click on the link (other than the current user) will not be
able to access the interview answers unless [`multi_user`] is set to
`True`.  This is because interviews are encrypted on the server by
default.  Setting [`multi_user`] to `True` disables this encryption.
Note that the communication between **docassemble** and the browser
will always be encrypted if the site is configured to use [HTTPS].
The server-side encryption merely protects against the scenario in
which the server running **docassemble** is compromised.

## <a name="interview_url_as_qr"></a>interview_url_as_qr()

Like `interview_url()`, except it inserts into the markup a [QR code]
linking to the interview.  The resulting [QR code] can be used to pass
control from a web browser or a paper handout to a mobile device.

## <a name="interview_url_action"></a>interview_url_action()

`interview_url_action()` is like [`interview_url()`], except that it
also has the effect of running [`url_action()`].  You will want to use
this instead of [`url_action()`] if you want the user to be able to
share the URL with other people, or run it unattended.

{% include side-by-side.html demo="interview_url_action" %}

Note the following about this example:

* [`multi_user`] must be set to `True`.  This disables server-side
  encryption of answers.  This is necessary because the encryption
  uses the user's password, and the password should not be embedded in
  a URL.
* [`initial`] code also runs [`process_action()`], which indicates when
  the action (in this case, `check_update_status`) should be done
  relative to other [`initial`] and [`mandatory`] code.

## <a name="interview_url_action_as_qr"></a>interview_url_action_as_qr()

Like `interview_url_action()`, except it inserts into the markup a
[QR code] linking to the interview, with the specified actions.

Note that there is a limit to the number of characters a [QR code] can
hold, and you might run up against this limit if you try to add too
many arguments to the URL.

## <a name="action_arguments"></a>action_arguments()

The `action_arguments()` function returns a dictionary with any
arguments that were passed when the user clicked on a link generated
by [`url_action()`] or [`interview_url_action()`].

## <a name="action_argument"></a>action_argument()

The `action_argument()` function is like [`action_arguments()`],
except that it returns the value of a given argument.

For example, if you formed a URL with:

{% highlight yaml %}
---
question: |
  The total amount of your bill is ${ currency(bill + tip) }.
subquestion: |
  [Tip your waiter $10](${ url_action('tip', amount=10)}).

  [Tip your waiter $20](${ url_action('tip', amount=20)}).
---
{% endhighlight %}

Then you could retrieve the value of `amount` by doing:

{% highlight yaml %}
---
event: tip
code: |
  tip = action_argument('amount')
---
{% endhighlight %}

## <a name="qr_code"></a>qr_code()

The `qr_code()` function allows you to include the `[QR ...]` [markup] statement
using [Python].

These two questions are equivalent:

{% highlight yaml %}
---
question: |
  Here is a QR code.
subquestion: |
  Go to Google:

  ${ qr_code('http://google.com', width='200px') }

  Or go to Yahoo:

  ${ qr_code('http://yahoo.com') }
sets: qr_example
---
{% endhighlight %}

{% highlight yaml %}
---
question: |
  Here is a QR code.
subquestion: |
  Go to Google:

  [QR http://google.com, 200px]

  Or go to Yahoo:

  [QR http://yahoo.com]
sets: qr_example
---
{% endhighlight %}

## <a name="url_of"></a>url_of()

This function returns a URL to a file within a **docassemble**
package.

For example, you might have PDF files associated with your interview.
You would keep these in the `data/static` directory of your package,
and you would refer to them by writing something like:

{% highlight yaml %}
---
mandatory: true
question: You are done.
subquestion: |
  To learn more about this topic, read
  [this brochure](${ url_of('docassemble.mycompany:data/static/brochure.pdf') }).
---
{% endhighlight %}

# <a name="background"></a>Background processes

If you include code in your interview that takes a long time to run,
such as code that looks up information in an on-line database, the
screen will take a long time to load and the user may think that the
application has "crashed" when it is actually just working normally.

To get around this problem, **docassemble** allows interview authors
to run code in "background processes."  While the user is answering
other questions, or looking at a user-friendly screen that instructs
the user to be patient, the **docassemble** server can be hard at work
carrying out a variety of time-consuming tasks for the user.

These processes can even operate in parallel.  For example, if your
interview searches the user's name in four different on-line
databases, all of these searches can be carried out simultaneously,
which will return a result to the user much faster than if the
searches were carried out one after the other.

**docassemble** runs these background tasks using [Celery], a
"distributed task queue" system.  [Celery] is highly scalable.  If you
are running **docassemble** on a single server and an interview starts
100 tasks at the same time, [Celery] will will queue the tasks and
handle them in order, working on several of them at a time.  And if
background tasks are particularly important for your application, you
can install [multiple servers] dedicated to handling these tasks.

## <a name="background_action"></a>background_action()

{% include side-by-side.html demo="background_action" %}

Starting a background process involves calling the
`background_action()` function.

{% highlight yaml %}
---
code: |
  the_task = background_action('bg_task', None, additional=value_to_add)
---
{% endhighlight %}

In this example, the first argument to [`background_action()`],
`bg_task`, is the name of an [action](#actions) available in the
interview that contains the time-consuming code you want to run.

The second argument to [`background_action()`] indicates how the
result of the action should be communicated to the user.  In this
case, `None` means no communication (more on this
[below](#ui_notification)).

The keyword argument, `additional`, is an argument that is passed to
that action (which can be retrieved with [`action_argument()`]).  The
parameters to `background_action()` should be familiar if you have
ever used [`url_action()`].  You can have as many keyword arguments as
you want, called anything you want, or you can have no keyword
arguments at all.

The `background_action()` function returns an object that represents a
[Celery] "task."  In this example, the object is saved to a variable
called `the_task`.  It can be used in the following ways:

* `the_task.ready()` returns `True` if the task has been completed
  yet, and `False` if not.
* `the_task.get()` returns the result of the task, if available.

Even if the `bg_task` action takes a long time to finish, the
`background_action()` function will always return a response right
away.  The `bg_task` action will run in the background, independently
of whatever goes on between the user and the web application.

[Celery] will start trying to run the the `bg_task` action as soon as
possible after `background_action()` is called.  If a lot of other
tasks are already running, the task will go into a queue and will be
run as soon as a [Celery] "worker" is available.

The background action runs much like other [actions] do: via the
[`process_action()`] function.  If you call `background_action()` in
your interview, make sure that you have included [`initial`] code that
calls [`process_action()`], as in the example above.  This is
important because it gives interview authors the ability to determine
the context in which the [`process_action()`] function is called.  For
example, you might want to call [`set_info()`] first to declare the
role of the user.

The code in a background action can use the [`user_logged_in()`],
[`user_has_privilege()`], and [`user_info()`] functions to determine
information about the current user (i.e. the user who caused the
`background_action()` function to be called).  In this respect,
background actions are different from [scheduled tasks], which always
run as the special "[cron user]."  In addition, background tasks are
different from [scheduled tasks] in that you can run background tasks
regardless of whether [`multi_user`] is set to `True` or `False`.

{% highlight yaml %}
---
event: bg_task
code: |
  # This is where time-consuming code would go
  background_response(553 + action_argument('additional'))
---
{% endhighlight %}

Any changes made to variables by a background action will not be
remembered after the action finishes.  You need to use
[`background_response()`] or [`background_response_action()`]
(discussed below) in order to communicate a result.  This limitation
exists because background actions are intended to run at the same time
the user is answering questions in the interview.  If the background
process starts at 3:05 p.m. and finishes at 3:10 p.m., but the user
answers five questions between 3:05 p.m. and 3:10 p.m., the user's
changes would be overwritten if the background process saved its
changes at 3:10 p.m.

## <a name="background_response"></a>background_response()

The `background_response()` function allows you to return information
from a background action to the interview.  The information is
accessed by using the `.get()` method on the "task" that was created.

For example, in the above example, the task is created like this:

{% highlight python %}
the_task = background_action('bg_task', None, additional=value_to_add)
{% endhighlight %}

There is now a variable `the_task` in the interview, which is used to
keep track of the status of the `bg_task` action, which is running in
the background.

The `bg_task` action does not change any of the variables in the
interview, but it passes its result back to the interview using
`background_response()`.

{% highlight python %}
background_response(553 + action_argument('additional'))
{% endhighlight %}

The response value is the sum of 553 and whatever number was provided
in the `additional` parameter.  This value is not saved to any
variable.  (Even if the background action tried to make a change to
variables in the interview's dictionary, those changes would be
forgotten once the action completes.)

The interview can retrieve the response value by calling the `.get()`
method on the `the_task` variable.  For example,

{% highlight yaml %}
question: |
  The answer is ${ the_task.get() }.
{% endhighlight %}

## <a name="background_response_action"></a>background_response_action()

It is possible for long-running tasks to save information to the
interview's dictionary, but they need to do so by sending the
information to a separate "action," the purpose of which is to save
the information to the interview's dictionary.  Information can be
passed to the action as arguments.

{% include side-by-side.html demo="background_action_with_response_action" %}

In this example, the action that runs in the background is `bg_task`
and the action that changes the interview's dictionary is `bg_resp`.

{% highlight yaml %}
---
event: bg_task
code: |
  value = 553 + action_argument('additional')
  background_response_action('bg_resp', ans=value)
---
event: bg_resp
code: |
  answer = action_argument('ans')
---
{% endhighlight %}

The `bg_task` action finishes by calling
`background_response_action('bg_resp', ans=value)`.
(`background_response_action()`, like other functions including
[`background_response()`], [`message()`], [`command()`], and
[`response()`], tells **docassemble** to stop whatever it is doing.
**docassemble** will not process any code that follows
`background_response_action()`.)

The first argument to `background_response_action()` is the name of
the action to be run, and the remainder of the arguments are keyword
arguments that are provided to the action.  In the above example, The
`bg_resp` action retrieves the argument `ans` and changes the variable
`answer` in the interview's dictionary to the value of `ans`.

The idea here is that `bg_task` is a long-running task, while
`bg_resp` is a short-running task devoted only to saving the results
of the long-running task.

When the code for the `bg_resp` action runs, it runs separately from
the `bg_task` action.  If `bg_task` changes a variable in the
interview's dictionary, the `bg_resp` action will not be able to see
those changes.  The only way the `bg_task` action can send information
to the `bg_resp` action is by passing arguments to it via the
`background_response_action()` function.

In computer science terminology, the `bg_resp` action is similar to a
[callback function].

These [background processes] are designed to run in the background
while the user is still using the application.  If the user closes the
web browser, the actions will still continue to run.

## <a name="ui_notification"></a>Communicating results to the user interface

**docassemble** can automatically alert the user when a background job
finishes.  There are three ways this can be done.

The first way is to "flash" a message at the top of a user's screen.

The second way is to cause the user's screen to reload the interview
when the job finishes.  If you used [`background_response_action()`]
to change the interview's dictionary on the basis of work done by the
background process, then the user may see a different screen after the
interview reloads.  However, it is important to be mindful of the
users' perspective when using this feature; you would not want to
annoy users by refreshing their screens while they are in the middle
of entering information.

The third way is to cause the user's browser to run [Javascript] code
produced by your background process.

You can cause these responses by setting the second argument to
[`background_action()`] to `'flash'`, `'refresh'`, or `'javascript'`.
Setting the second argument to `None` means that no notification of
any kind will be sent to the user's browser.

In the following example, the value provided to
[`background_response()`] ("The answer is (some number)."), is
"flashed" at the top of the screen.

{% include side-by-side.html demo="background_action_flash" %}

You can also "flash" messages when you use
[`background_response_action()`] to run a separate [action](#actions)
that saves changes to the interview's dictionary.  In your action that
sets variables, conclude your code with a call to
[`background_response()`] containing the message you want to flash.

{% include side-by-side.html demo="background_response_action_flash" %}

In the next example, when the background process finishes, the user's
screen refreshes.  Since the [`background_response_action()`] function
was used to save the result of the background process to a variable
(`answer`), the result can be displayed by referring to `${ answer }`.

{% include side-by-side.html demo="background_action_refresh" %}

The next example is like the first, except the notification takes
place through [Javascript] code created by the background process,
which in this case uses the built-in [Javascript] function
[`alert()`] to send a message to the user.

{% include side-by-side.html demo="background_action_javascript" %}

Note that the user interface does not respond immediately when the
background task finishes.  The user's browser polls the server every
six seconds (edit [`checkin interval`] to adjust this).  Therefore,
users may experience up to a six-second delay after the background
process finishes before they receive notification.

## Comparison with scheduled tasks

**docassemble** also has a [scheduled tasks] feature, which is similar
to the [background processes] feature in that the code runs in the
background, without any direct interaction with the user.  The
[scheduled tasks] are different in that they are triggered at monthly,
weekly, daily, or hourly intervals, rather than being triggered by the
user.

Another difference is that [scheduled tasks] are always run by a
[special user], and this requires that server-side encryption be
disabled in the interview.  These restrictions do not apply to
[background processes].

# Geographic functions

## <a name="map_of"></a>map_of()

The `map_of()` function inserts a Google Map into question text.  (It
does not work within documents.)  The arguments are expected to be
**docassemble** [objects].  Different objects are mapped differently:

* [`Address`] objects: if an [`Address`] object is provided as an argument
  to `map_of()`, a map marker will be placed at the geolocated
  coordinates of the address.  The description of the address will be
  the address itself.
  * _Technical details_: if the object is called `address`, the marker
    will be placed at the coordinates `address.location.latitude` and
    `address.location.longitude`.  (The attribute `address.location`
    is a [`LatitudeLongitude`] object.)  The description of the marker
    will be set to `address.location.description`.  These fields are
    set automatically during the geolocation process, which will take
    place the first time **docassemble** runs `map_of()`, if it has
    not taken place already.  The marker icon can be customized by
    setting `address.icon`.
* [`Organization`] objects: map markers will be placed at the
  locations of each of the organization's offices.  For example, if
  the object name is `company`, markers will be placed on the map for
  each address in `company.office` (which is a list of [`Address`]es).
  The icon for the `i`th office will be `company.office[i].icon`, or,
  if that is not defined, it will be `company.icon` if that is
  defined.  The description of each marker will be the organization's
  name (`company.name.full()`) followed by
  `company.office[i].location.description`.
* [`Person`] objects: a map marker will be placed at the person's
  address.  The description will be the person's name, followed by the
  address.  The marker icon can be customized by setting `person.icon`
  (for a [`Person`] object called `person`).  If the [`Person`] object
  is the user, the default icon is a blue circle.

## <a name="location_known"></a>location_known()

Returns `True` or `False` depending on whether **docassemble** was
able to learn the user's GPS location through the web browser.

See [`track_location`] and [`LatitudeLongitude`] for more information
about how **docassemble** collects information about the user's
location.

## <a name="location_returned"></a>location_returned()

Returns `True` or `False` depending on whether an attempt has yet been
made to transmit the user's GPS location from the browser to
docassemble.  Will return true even if the attempt was not successful
or the user refused to consent to the transfer.

See [`track_location`] and [`LatitudeLongitude`] for
more information about how **docassemble** collects information about
the user's location.

## <a name="user_lat_lon"></a>user_lat_lon()

Returns the user's latitude and longitude as a tuple.

See [`track_location`] and [`LatitudeLongitude`] for more information
about how **docassemble** collects information about the user's
location.

# Functions for managing global variables

If you try writing your own functions, you will learn that functions
do not have access to all of the variables in your interview.
Functions only know the variables you pass to them.

If your functions need to know background information about the
interview, but you do not want to have to pass a lot of variables to
every function you call, you can use "global" variables.

You set "global" variables in **docassemble** by calling [`set_info()`]
and your retrieve them by calling [`get_info()`].  Note that
**docassemble** will forget the values of these variables every time
the screen loads, so you will have to make sure they are set by
setting them in [`initial`] code, which runs every time the screen
loads.

## <a name="set_info"></a>set_info()

This function is used to store information for later retrieval by
`get_info()`.  You pass it one or more [keyword arguments]:

{% highlight yaml %}
---
initial: true
code: |
  set_info(interview_type='standard')
---
{% endhighlight %}

Some of the [functions] and [methods] of [`docassemble.base.util`]
will behave differently depending on who the interviewee is and what
the interviewee's role is.  For example, if `trustee` is an object of
the class [`Individual`], and you call `trustee.do_question('have')`,
the result will be "do you have" if if the interviewee is the trustee,
but otherwise the result will be "does Fred Jones have" (or whatever
the trustee's name is).

In order for the [`docassemble.base.util`] module to know this
background information, you need to include an [`initial`] code block
(or a [`default role`] block containing [`code`]) that:

1. Defines `user` as an object of the class [`Individual`];
2. Defines `role` as a text value (e.g., `'advocate'`); and
3. Calls `set_info(user=user, role=role)`.

For example, this is how [`basic-questions.yml`] does it:

{% highlight yaml %}
---
objects:
  - client: Individual
  - advocate: Individual
  # etc.
---
default role: client
code: |
  if user_logged_in() and user_has_privilege('advocate'):
    user = advocate
    role = 'advocate'
  else:
    user = client
    role = 'client'
  set_info(user=user, role=role)
---
{% endhighlight %}

(See [initial blocks] for an explanation of [`objects`] and [`default
role`].  See the [roles] section for an explanation of how user roles
work in **docassemble**.)

## <a name="get_info"></a>get_info()

This function is used to retrieve information passed to
`set_info()`.

For example, if you passed `interview_type` as a [keyword argument] to
`set_info()`, you can retrieve the value in your [Python module] by
doing:

{% highlight python %}
from docassemble.base.util import *

class Recipe(DAObject):
    def difficulty_level(self):
        if get_info('interview_type') == 'standard':
            #etc.
{% endhighlight %}

If the item was never set, `get_info()` will return `None`.

## <a name="interface"></a>interface()

The `interface()` function returns `'web'` if the user is accessing
the interview through a web browser and `'sms'` if the user is using
[SMS].

Sometimes interviews are accessed by background processes.
`interface()` will return `'cron'` if the interview is being accessed
by a [scheduled task], and will return `'worker'` if it is being
accessed by a [background process].

You might want to use this function to provide special instructions to
users depending on the way they access the interview.  For example,
the following will show a special instruction screen to users who are
accessing the interview through [SMS].

{% highlight yaml %}
---
mandatory: true
code: |
  if interface() == 'sms':
    sms_instructions
---
question: |
  Instructions
subquestion: |
  To leave the interview, type exit.
field: sms_instructions
---
{% endhighlight %}

You can also use `interface()` to distinguish between actual user
requests and requests that originate from [background processes].

{% highlight yaml %}
---
code: |
  request_counter = 0
---
initial: true
code: |
  if interface() in ['sms', 'web']:
    request_counter += 1
---
{% endhighlight %}



## <a name="user_logged_in"></a>user_logged_in()

The `user_logged_in()` function returns `True` if the user is logged
in, and otherwise returns `False`.

## <a name="user_privileges"></a>user_privileges()

The `user_privileges()` function returns the user's privileges as a
list.  If the user is not logged in, the result is `['user']`.  If the
user is a "user" as well as a "customer," the result is
`['user', 'customer']`.  If the interview is running a
[scheduled task] in the background, the result is `['cron']`.

## <a name="user_has_privilege"></a>user_has_privilege()

The `user_has_privilege()` function returns `True` if the user has any
of the given privileges, and `False` otherwise.  For example, if the
user has the privilege of "customer," `user_has_privilege('customer')`
will return `True`.  A list can also be provided, in which case `True`
will be returned if the user has any of the given privileges.  For
example, if the user has the privilege of "developer",
`user_has_privilege(['developer', 'admin'])` will return `True`.

## <a name="user_info"></a>user_info()

The `user_info()` function will return an object with the following
attributes describing the current user:

* `first_name`
* `last_name`
* `email`
* `country` (will be an official [country code] like `us`)
* `subdivision_first` (e.g., state)
* `subdivision_second` (e.g., county)
* `subdivision_third` (e.g., municipality)
* `organization` (e.g., company or non-profit organization)

These attributes are set by the user on the [Profile page].

For example:

{% highlight yaml %}
---
question: |
  Your e-mail address is ${ user_info().email }.  Is that
  the best way to reach you?
yesno: email_is_best
---
{% endhighlight %}

# Language and locale functions()

These functions access and change the active language and locale.  See
[language support] for more information about these features of
**docassemble**.

## <a name="get_language"></a>get_language()

If the language is set to English, `get_language()` returns `en`.

## <a name="set_language"></a>set_language()

This sets the language that will be used in the web application and in
language-specific functions of **docassemble**.  It does not change
the active [Python locale].  See `update_locale()` for information on
changing the [Python locale].

If you need to set the language to something other than the language
set in the [configuration], you need to call `set_language()` within
[`initial`] code.  For example:

{% highlight yaml %}
---
modules:
  - docassemble.base.util
---
initial: true
code: |
  set_language(user_language)
---
question: |
  What language do you speak? (¿Qué idioma habla?)
field: user_language
choices:
  - English: en
  - Español: es
---
{% endhighlight %}

Using the optional `dialect` keyword argument, you can also set the
dialect of the language.  The dialect is relevant only for the
text-to-speech engine.  For example:

{% highlight yaml %}
---
initial: true
code: |
  set_language('en', dialect='au')
---
{% endhighlight %}

This will set the language to English, and will instruct the
text-to-speech engine to use an Australian dialect.  (The dialect is
relevant only for the text-to-speech engine, which is controlled by
the [special variable `speak_text`].)

## <a name="get_dialect"></a>get_dialect()

Returns the current dialect, as set by the `dialect` keyword argument
to the `set_language()` function.

## <a name="get_country"></a>get_country()

Returns the current country as a two-digit country code.  The default
country is `'US'`, unless a different default is set using the
[`country` configuration setting].

## <a name="set_country"></a>set_country()

Sets the current country.  Expects a two-digit, uppercase country code
such as `'US'`, `'GB'`, or `'DE'`.

## <a name="get_locale"></a>get_locale()

If the locale is set to `US.utf8`, `get_locale()` returns `US.utf8`.

## <a name="set_locale"></a>set_locale()

If you run `set_locale('FR.utf8')`, then `get_locale()` will return
`FR.utf8`, but the actual [Python locale] will not change
unless you run `update_locale()`.

## <a name="update_locale"></a>update_locale()

Running `update_locale` will change the [Python locale] based on the
current language and locale settings.

For example, if you run:

{% highlight python %}
import docassemble.base.util
docassemble.base.util.set_language('fr')
docassemble.base.util.set_locale('FR.utf8')
docassemble.base.util.update_locale()
{% endhighlight %}

then the [Python locale] will be changed to `fr_FR.utf8`.

Running `update_locale()` is necessary in order to affect the behavior
of functions like [`currency()`] and [`currency_symbol()`].

Note that changes to the locale are not thread-safe.  This means that
there is a risk that between the time **docassemble** runs
`update_locale()` and the time it runs [`currency_symbol()`], another
user on the same server may cause **docassemble** to run
`update_locale()` and change it to the wrong setting.

If you want to host different interviews that use different locale
settings on the same server (e.g., to format a numbers as 1,000,000 in
one interview, but 1.000.000 in another), you will need to make sure
you run the **docassemble** web server in a multi-process,
single-thread configuration.  (See [installation] for instructions on
how to do that.)  Then you would need to begin each interview with
[`initial`] code such as:

{% highlight yaml %}
---
initial: true
code: |
  import docassemble.base.util
  docassemble.base.util.set_language('fr')
  docassemble.base.util.set_locale('FR.utf8')
  docassemble.base.util.update_locale()
---
{% endhighlight %}

## <a name="countries_list"></a><a name="country_name"></a>countries_list() and country_name()

The `countries_list()` function returns a list of dictionaries, where
each dictionary contains a single key-value pair mapping a two-letter,
capitalized country abbreviation to the name of the country (in
English).  This function is primarily useful when asking a user to
specify his or her country.

The `country_name()` function returns the name of a country (in
English) based on the two-letter, capitalized country abbreviation.

{% include side-by-side.html demo="country" %}

The data come from the [`pycountry` package].

When working with countries, it is a good idea to store country names
in this two-letter, capitalized format.  The country code is used by
the [`send_sms()`] function to determine the appropriate universal
formatting of phone numbers.

# Access time functions

Internally, **docassemble** keeps track of the last time the interview
was accessed.  The following functions retrieve information about
access times.  These functions are particularly useful in
[scheduled tasks].

## <a name="last_access_time"></a>last_access_time()

`last_access_time()` returns a [`datetime`] object containing the last
time the interview was accessed by a user other than the special
[cron user].  The time is expressed in [UTC time] without a time zone.

Optionally, a role name or a list of role names can be provided.  In
this case, the function will return the latest access time by any user
holding one of the [roles].

* `last_access_time('client')`: returns the last time a user with the
  role of `client` accessed the interview.
* `last_access_time('advocate')`: returns the last time a user with
  the role of `advocate` accessed the interview.
* `last_access_time(['advocate', 'admin'])`: returns the last time a
  user with the role of `advocate` or `admin` accessed the interview.

By default, `last_access_time()` will ignore interview access by the
[cron user].  However, if you do not wish to ignore access by the
[cron user], you can call `last_access_time()` with the optional
keyword argument `include_cron` equal to `True`:

* `last_access_time(include_cron=True)`: returns the last time any
  user, including the [cron user] if applicable, accessed the
  interview.

## <a name="last_access_days"></a>last_access_days()

The function `last_access_days()` works just like
[`last_access_time()`], except that it returns the number of days
(including fractional days) between the current time and the last
access time.

## <a name="last_access_hours"></a>last_access_hours()

Like [`last_access_days()`], except returns hours.

## <a name="last_access_minutes"></a>last_access_minutes()

Like [`last_access_days()`], except returns minutes.

## <a name="last_access_delta"></a>last_access_delta()

The function `last_access_delta()` works just like
[`last_access_time()`], except that it returns a
[`datetime.timedelta`] object giving the difference between the
current time and the last access time.

# Functions for working with dates and times

## <a name="month_of"></a><a name="day_of"></a><a name="year_of"></a>month_of(), day_of(), and year_of()

These functions read a date and provide the parts of the date.

{% include side-by-side.html demo="date-parts" %}

The `month_of` function has an optional setting: if called as, e.g.,
`month_of(some_date, as_word=True)`, it will return the month as a
word (according to the current language and locale).

## <a name="format_date"></a>format_date()

The `format_date()` function takes as input a date, which could be
written in any format, and returns the date formatted appropriately
for the current language.

For example:

* `format_date("10/31/2016")` returns `October 31, 2016`.
* `format_date("2016-04-01")` returns `April 1, 2016`.
* `format_date("March 3, 2016")` returns `March 3, 2016`.
* `format_date('April 5, 2014', format='full')` returns `Saturday, April 5, 2014`.
* `format_date('April 5, 2014', format='short')` returns `4/5/14`.
* `format_date('April 5, 2014', format='M/d/yyyy')` returns `4/5/2014`.
* `format_date('April 5, 2014', format='MM/dd/yyyy')` returns
  `04/05/2014`.

For more information about how to specify date formats, see the
documentation for
[babel.dates](http://babel.pocoo.org/en/latest/api/dates.html).  The
`format` argument, which defaults to `long`, is passed directly to the
`babel.dates.format_date()` function.

## <a name="format_time"></a>format_time()

The `format_time()` function works just like [`format_date()`], except
it returns a time, rather than a date.

For example:

* `format_time("04:01:23")` returns `4:00 AM`.

For more information about how to specify time formats, see the
documentation for
[babel.dates](http://babel.pocoo.org/en/latest/api/dates.html).  The
`format` argument, which defaults to `short`, is passed directly to
the `babel.dates.format_time()` function.

## <a name="today"></a>today()

{% include side-by-side.html demo="today" %}

Returns today's date in long form according to the current locale
(e.g., `March 31, 2016`).  It is like `format_date()` in that it
accepts the optional keyword argument `format`.  It also takes an
optional keyword argument `timezone`, which refers to one of the time
zone names in [`timezone_list()`].

For example:

* `today(format='M/d/yyyy')`
* `today(timezone='US/Eastern')`

If the `timezone` is not supplied, the default time zone will be
used.

## <a name="timezone_list"></a>timezone_list()

{% include side-by-side.html demo="timezone-list" %}

The `timezone_list()` function returns a list of time zones that the
other date-related functions in **docassemble** understand.  The list
is generated from the [`pytz`] module.  The primary purpose of this
function is to include in a multiple choice question.

Note that the items in this list are strings, like `America/New_York`.

## <a name="get_default_timezone"></a>get_default_timezone()

{% include side-by-side.html demo="get-default-timezone" %}

The `get_default_timezone()` function returns the default timezone.
This is the time zone of the server, unless the default timezone is
set using the [`timezone` configuration].

## <a name="as_datetime"></a>as_datetime()

{% include side-by-side.html demo="as-datetime" %}

The `as_datetime()` function expresses a date as a [`datetime`]
object with a time zone.  It takes an optional keyword argument,
`timezone`, which will override the default time zone.

## <a name="current_datetime"></a>current_datetime()

{% include side-by-side.html demo="current-datetime" %}

The `current_datetime()` function returns the current date and time as
a [`datetime`] object.  It takes an optional keyword argument,
`timezone`, which will override the default time zone.

## <a name="date_difference"></a>date_difference()

{% include side-by-side.html demo="date-difference" %}

The `date_difference()` function returns the difference between two
dates as an object with attributes that express the difference using
different units.  The function takes two keyword arguments: `starting`
and `ending`.  If either is omitted, the `current_datetime()` is used
in its place.

If you do `date_difference(starting=a, ending=b)`, then if date `a` comes
before date `b`, the resulting values will be positive.  But if date
`b` comes before date `a`, the values will be negative.

For example, if you set `z = date_difference(starting='1/1/2015',
ending='1/3/2015')`, then:

* `z.weeks` returns `0.2857142857142857`.
* `z.days` returns `2.0`.
* `z.hours` returns `48.0`.
* `z.minutes` returns `2880.0`.
* `z.seconds` returns `172800`.
* `z.delta` returns a [`datetime.timedelta`] object representing the
  difference between the times.

Dates without time zones are localized into the default time zone
before the calculation takes place.  You can supply the optional
keyword argument `timezone` to use a different time zone.

## <a name="date_interval"></a>date_interval()

{% include side-by-side.html demo="date-interval" %}

The `date_interval()` function can be used to perform calculations on
a date.  For example,

* `current_datetime() + date_interval(days=1)` represents 24 hours in
  the future.
* `current_datetime() - date_interval(years=1)` represents one year in
  the past.

The available keyword arguments are:

* `years`
* `months`
* `days`
* `weeks`
* `hours`
* `minutes`
* `seconds`
* `microseconds`

This function is a direct wrapper around
[`dateutil.relativedelta.relativedelta`].

# <a name="phone"></a>Functions for working with phone numbers

## <a name="phone_number_in_e164"></a>phone_number_in_e164()

The `phone_number_in_e164()` function takes a phone number and formats
it into [E.164] format.  It takes an optional keyword argument
`country` that is used to determine the country code for the phone
number.  If `country` is not provided, the [`get_country()`] function
is used to determine the applicable country.

## <a name="phone_number_is_valid"></a>phone_number_is_valid()

The `phone_number_is_valid()` function takes a phone number and
returns `True` or `False` depending on whether the phone number is
valid.  It takes an optional keyword argument `country` that is used
to determine the country whose phone number standards should be used
to determine the validity of the phone number.  If `country` is not
provided, the [`get_country()`] function is used to determine the
applicable country.

# <a name="tasks"></a>Functions for tracking tasks

These are helpful functions for keeping track of whether certain tasks
have been performed.  For example, if your interview sends an e-mail
to the user about something, but you want to avoid sending the e-mail
more than once, you can give the "task" a name and use these functions
in your code to make sure your interview only sends the e-mail if it
has never been successfuly sent before.

Instead of using these functions, you could use your own variables to
keep track of whether tasks have been carried out or not.  These
functions do not do anything besides keep track of information.  A
good reason to use these functions is to increase the readability of
your code.

## <a name="task_performed"></a>task_performed():

The `task_performed()` function returns `True` if the task has been
performed at least once, otherwise `False`.

For example, `task_performed('application_for_assistance')` will
return `False` until
`mark_task_as_performed('application_for_assistance')` is called.

## <a name="task_not_yet_performed"></a>task_not_yet_performed():

The `task_not_yet_performed()` function returns `True` if the task has
never been performed, otherwise `False`.  It is simply the opposite of
[`task_performed()`].

## <a name="mark_task_as_performed"></a>mark_task_as_performed():

The `mark_task_as_performed()` function increases by 1 the number of
times the task has been performed.

For example, if `task_performed('send_reminder')` is `False`, and then
you call `mark_task_as_performed('remind_user')`, then
`task_performed('send_reminder')` will now return `True`, and
`times_task_performed('remind_user')` will return `1`.

If you call `mark_task_as_performed('remind_user')` again,
`task_performed('send_reminder')` will still return `True`, and
`times_task_performed('remind_user`) will now return `2`.

## <a name="times_task_performed"></a>times_task_performed():

The `times_task_performed()` function returns the number of times the
task has been performed (i.e., the number of times
[`mark_task_as_performed()`] is called).  If the task has never been
performed, `0` is returned.

## <a name="set_task_counter"></a>set_task_counter():

The `set_task_counter()` function allows you to manually set the
number of times the task has been performed.
`set_task_counter('remind_user', 0)` sets the counter of the
`remind_user` task to zero, which means that
`task_performed('remind_user')` would subsequently return `False`.

# Simple translation of words

## <a name="word"></a>word()

`word()` is a general-purpose translation function that is used in the
code of the web application to ensure that the text the user sees is
translated into the user's language.

`word("fish")` will return `fish` unless
`docassemble.base.util.update_word_collection()` has been used to
define a different translation for the current language.

The following [Python interpreter] session demonstrates how it works:

{% highlight python %}
>>> from docassemble.base.util import *
>>> set_language('es')
>>> word("fish")
u'fish'
>>> import docassemble.base.util
>>> docassemble.base.util.update_word_collection('es', {'fish': 'pescado'})
>>> word("fish")
u'pescado'
>>> set_language('en')
>>> word("fish")
u'fish'
{% endhighlight %}

In your own [Python] code you may wish to use `word()` to help make
your code multi-lingual.

It is not a good idea to call
`docassemble.base.util.update_word_collection()` in interviews.  You
can use it in [Python] modules, but keep in mind that the changes you
make will have global effect within the [WSGI] process.  If other
interviews on the server define the same word translations for the
same language using `docassemble.base.util.update_word_collection()`,
the module that happened to load last will win, and the results could
be unpredictable.

The best practice is to load translations at the server level by using
the [`words`] [configuration] directive to load translations from one or
more [YAML] files.  This causes **docassemble** to call
`docassemble.base.util.update_word_collection()` at the time the
server is initialized.

# Language-specific functions

These functions behave differently according to the language and
locale.  You can write functions for different languages, or reprogram
the default functions, by calling
`docassemble.base.util.update_language_function()`.

## <a name="capitalize"></a>capitalize()

{% include side-by-side.html demo="capitalize" %}

If `favorite_food` is defined as "spaghetti marinara," then
`capitalize(favorite_food)` will return `Spaghetti marinara`.
This is often used when a variable value begins a sentence.  For example:

{% highlight yaml %}
question: |
  ${ capitalize(favorite_food) } is being served for dinner.  Will you
  eat it?
yesno: user_will_eat_dinner
{% endhighlight %}

There is also the [`title_case()`] function, which is described below.

## <a name="comma_and_list"></a>comma_and_list()

If `things` is a [Python list] with the elements
`['lions', 'tigers', 'bears']`, then:

* `comma_and_list(things)` returns `lions, tigers, and bears`.
* `comma_and_list(things, oxford=False)` returns `lions, tigers and bears`.
* `comma_and_list('fish', 'toads', 'frogs')` returns `fish,
toads, and frogs`.
* `comma_and_list('fish', 'toads')` returns `fish and toads`
* `comma_and_list('fish')` returns `fish`.

## <a name="comma_list"></a>comma_list()

If `things` is a [Python list] with the elements
`['lions', 'tigers', 'bears']`, then `comma_list(things)` will return
`lions, tigers, bears`.

## <a name="currency"></a>currency()

If the locale is `US.utf8`, `currency(45.2)` returns `$45.20`.

`currency(45)` returns `$45.00`, but `currency(45, decimals=False)`
returns `$45`.

With `decimals` unset or equal to `True`, this function uses the
`locale` module to express the currency.  However, `currency(x,
decimals=False)` will simply return [`currency_symbol()`] followed by
`x` formatted as an integer, which might not be correct in your
locale.  This is due to a limitation in the [locale module].  If the
`currency` function does not meet your currency formatting needs, you
may want to define your own.

## <a name="currency_symbol"></a>currency_symbol()

If the locale is `US.utf8`, `currency_symbol()` returns `$`.

The locale can be set in the [configuration] or through the
[`set_locale()`] function.

If you set [`currency_symbol`] in the [configuration], then
`currency_symbol()` returns the symbol specified there, and does not
use the locale to determine the symbol.

## <a name="indefinite_article"></a>indefinite_article()

{% include side-by-side.html demo="indefinite-article" %}

The English language version of this function passes all arguments
through to the `article()` function of [pattern.en].

`indefinite_article('bean')` returns `a bean` and
`indefinite_article('apple')` returns `an apple`.

## <a name="nice_number"></a>nice_number()

{% include side-by-side.html demo="nice-number" %}

* `nice_number(4)` returns `four`
* `nice_number(10)` returns `ten`
* `nice_number(11)` returns `11`
* `nice_number(-1)` returns `-1`

This function can be customized by calling
`docassemble.base.util.update_nice_numbers()`.

## <a name="noun_plural"></a>noun_plural()

{% include side-by-side.html demo="noun-plural" %}

The English language version of this function passes all arguments
through to the `pluralize()` function of [pattern.en].

* `noun_plural('friend')` returns `friends`
* `noun_plural('fish')` returns `fish`
* `noun_plural('moose')` returns `mooses`

You can also pass a number as a second argument to the function.  If
the number is 1, the first argument will be returned, untouched;
otherwise, the pluralized version of the first argument will be returned.

* `noun_plural('friend', number_friends)` returns `friend` if
  `number_friends` is `1`, otherwise it returns `friends`.

## <a name="quantity_noun"></a>quantity_noun()

{% include side-by-side.html demo="quantity-noun" %}

This function combines [`nice_number()`] and [`noun_plural()`] into a
single function.  It will round the number to the nearest integer unless
you add the optional keyword argument `as_integer=False`.

* `quantity_noun(2, "apple")` returns `two apples`.
* `quantity_noun(1, "apple")` returns `one apple`.
* `quantity_noun(144, "apple")` returns `144 apples`.
* `quantity_noun(1.5, "apple")` returns `2 apples`.
* `quantity_noun(1.5, "apple", as_integer=False)` returns `1.5 apples`.

## <a name="noun_singular"></a>noun_singular()

The English language version of this function passes all arguments
through to the `singularize()` function of [pattern.en].

* `noun_singular('friends')` returns `friend`
* `noun_singular('fishes')` returns `fish`
* `noun_singular('mooses')` returns `moose`

You can also pass a number as a second argument to the function.  If
the number is 1, the singularized version of the first argument will
be returned.  Otherwise, the first argument will be returned,
untouched.

* `noun_singular('friends', number_friends)` return `friend` if
  `number_friends` is `1`, otherwise it returns `friends`.

## <a name="ordinal_number"></a>ordinal_number()

* `ordinal_number(8)` returns `eighth`.
* `ordinal_number(11)` returns `11th`.

This function can be customized with
`docassemble.base.util.update_ordinal_numbers()` and
`docassemble.base.util.update_ordinal_function()`.

## <a name="ordinal"></a>ordinal()

`ordinal(x)` returns `ordinal_number(x + 1)`.  This is useful when
working with indexes that start at zero.

## <a name="period_list"></a>period_list()

`period_list` returns a list within a list:

{% highlight python %}
[[12, "Per Month"], [1, "Per Year"], [52, "Per Week"],
[24, "Twice Per Month"], [26, "Every Two Weeks"]]
{% endhighlight %}

This is useful for using in `code` associated with periodic currency
amounts.

{% highlight yaml %}
---
modules:
  - docassemble.base.util
---
question: |
  Your income
subquestion: |
  You earn ${ currency(user_income_amount * user_income_period) }
  per year.
mandatory: true
---
question: |
  What is your income?
fields:
  - Income: user_income_amount
    datatype: currency
  - Period: user_income_period
    datatype: number
    code: |
      period_list()
---
{% endhighlight %}

([Try it out here]({{ site.demourl }}?i=docassemble.demo:data/questions/income.yml){:target="_blank"}.)

The text in the default `period_list()` function can be translated to
different languages using the
`docassemble.base.util.update_word_collection()` function.  If you
want to customizes the choices available, you can override the default
function by including something like the following in your
[Python module]:

{% highlight python %}
def my_period_list():
  return [[365, word("Per Day")], [52, word("Per Week")]]

docassemble.base.util.update_language_function('*', 'period_list', my_period_list)
{% endhighlight %}

## <a name="name_suffix"></a>name_suffix()

Like `period_list()`, except it represents common suffixes of
individual names.

Returns the following list:
{% highlight python %}
['Jr', 'Sr', 'II', 'III', 'IV', 'V', 'VI']
{% endhighlight %}

Here is a question that asks for the user's name with an optional
suffix:

{% highlight yaml %}
---
modules:
  - docassemble.base.util
---
question: |
  What is your name?
fields:
  - First Name: user.name.first
  - Middle Name: user.name.middle
    required: False
  - Last Name: user.name.last
  - Suffix: user.name.suffix
    required: False
    code: |
      name_suffix()
---
{% endhighlight %}

## <a name="title_case"></a>title_case()

{% include side-by-side.html demo="title-case" %}

`title_case("the importance of being ernest")` returns `The Importance
of Being Ernest`.

The default version of this function passes through all arguments to
the `titlecase()` function of the [titlecase] module.

There is also the `capitalize()` function, which is described above.

## <a name="verb_past"></a>verb_past()

{% include side-by-side.html demo="past-tense" %}

The English-language version of this function passes all arguments
through to the `conjugate()` function of [pattern.en].

* `verb_past('help', '3gp')` returns `helped` (third person past tense).
* `verb_past('help', 'ppl')` returns `helped` (plural past tense).

## <a name="verb_present"></a>verb_present()

The English-language version of this function passes through all
arguments to the `conjugate()` function of the [pattern.en].

* `verb_present('helped', '3sg')` returns `helps` (third person singular).
* `verb_present('helps', '1sg')` returns `help` (first person singular).
* `verb_present('helps', 'pl')` returns `help` (plural).

# Simple language functions

The following simple language functions all have the property that if
the optional argument `capitalize=True` is added, the resulting phrase
will be capitalized.

* `her('turtle')` returns `her turtle`.
* `her('turtle', capitalize=True)` returns `Her turtle`.
* `his('turtle')` returns `his turtle`.
* `a_in_the_b('cat', 'hat')` returns `cat in the hat`.
* `do_you('smoke')` returns `do you smoke`.
* `does_a_b('Fred', 'smoke')` returns `does Fred smoke`.
* `in_the('house')` returns `in the house`.
* `of_the('world')` returns `of the world`.
* `possessify('Fred', 'cat')` returns `Fred's cat`.
* `possessify_long('Fred', 'cat')` returns `the cat of Fred`.
* `the('apple')` returns `the apple`.

Note that unlike other functions in [`docassemble.base.util`], these
functions are *not* available for use within interviews.  If you do:

{% highlight yaml %}
modules:
  - docassemble.base.util
{% endhighlight %}

you will *not* be able to write things like `${ her('dog') }`.

These functions are intended to be used from within [Python modules], where you
can import them by doing:

{% highlight python %}
from docassemble.base.util import his, her
{% endhighlight %}

You can customize the functions for different languages:

{% highlight python %}
def her_fr(word, capitalize=False):
  if capitalize:
    return 'Sa ' + word
  else:
    return 'sa ' + word
docassemble.base.util.update_language_function('fr', 'her', her_fr)
{% endhighlight %}

Or, you can accomplish the same result with a handy function generator
from [`docassemble.base.util`]:

{% highlight python %}
docassemble.base.util.update_language_function('fr', 'her', docassemble.base.util.prefix_constructor('sa '))
{% endhighlight %}

# <a name="functions"></a>Miscellaneous functions

## <a name="static_image"></a>static_image()

{% include side-by-side.html demo="static_image" %}

Returns appropriate markup to include a static image.  If you know the
image path, you can just use the `[FILE ...]` [markup] statement.  The
`static_image()` function is primarily useful when you want to
assemble the image path using code.

The function takes an optional keyword argument "width" that will
affect the width of the image on the screen or page:

{% highlight python %}
static_image('docassemble.demo:crawling.png', width='2in'))
{% endhighlight %}

## <a name="get_config"></a>get_config()

Returns a value from the **docassemble** configuration file.  If the
value is defined, returns None.

See the explanation of this function in the
[configuration section]({{ site.baseurl }}/docs/config.html#get_config")
for more information.

## <a name="send_email"></a>send_email()

The `send_email()` function sends an e-mail using [Flask-Mail].  All
of its arguments are [keyword arguments], the defaults of which are:

{% highlight python %}
send_email(to=None, sender=None, cc=None, bcc=None, body=None, html=None, subject="", template=None, task=None, attachments=None)
{% endhighlight %}

This function is integrated with other classes in
[`docassemble.base.util`] and [`docassemble.base.core`].

* `to` expects a [list] of [`Individual`]s.
* `sender` expects a single [`Individual`].  If not set, the
  `default_sender` information from the [configuration] is used.
* `cc` expects a [list] of [`Individual`]s, or `None`.
* `bcc` expects a [list] of [`Individual`]s, or `None`.
* `body` expects text, or `None`.  Will set the plain text content of
  the e-mail.
* `html` expects text, or `None`.  Will set the (optional) [HTML]
  content of the e-mail.
* `subject` expects text, or `None`.  Will set the subject of the e-mail.
* `template` expects a [`DATemplate`] object, or `None`.  These
  templates can be created in an interview file using the `template`
  directive.  If this [keyword argument] is supplied, both the plain
  text and [HTML] contents of the e-mail will be set by converting the
  [Markdown] text of the template into [HTML] and by converting the
  [HTML] to plain text (using [html2text]).  In addition, the subject
  of the e-mail will be set to the subject of the template.  You can
  override any of these behaviors by manually specifying `body`,
  `html`, or `subject`.
* `task` expects the name of a [task].  If this argument is provided,
  and if sending the e-mail is successful, the task will be marked as
  having been performed (i.e., [`mark_task_as_performed()`] will be
  called).  Alternatively, you can handle this in your own code, but
  you might find it convenient to let the `send_email()` function
  handle it for you.
* `attachments` expects a [list] of [`DAFile`], [`DAFileList`], or
  [`DAFileCollection`] objects.  You can include:
  * Images generated by `signature` blocks (objects of class
  [`DAFile`]);
  * File uploads generated by including [fields] of `datatype: file` or
  `datatype: files` in a [`question`] (objects of class [`DAFileList`]);
  * [Documents] generated by [`attachments`] to a [`question`] for which a
  `variable` was provided (objects of class [`DAFileCollection`]).

It uses the `name` and `email` attributes of the listed [`Individual`]s
to form e-mail addresses.

`send_email()` returns `False` if an error prevented the e-mail from
being sent; otherwise it returns `True`.

See [configuration] for information about how to configure the mail
server that `send_email()` will use.

Here is an example:

{% highlight yaml %}
---
modules:
  - docassemble.base.util
---
objects:
  - user: Individual
---
mandatory: true
question: |
  So long, ${ user }!
subquestion: |
  % if success_sending_email:
  We sent an e-mail to your e-mail address.
  % else:
  Oops, for some reason we could not send an e-mail to you.
  % endif
---
question: |
  Please fill in the following information.
fields:
  - Your First Name: user.name.first
  - Your Last Name: user.name.last
  - Your E-mail: user.email
    datatype: email
  - A Picture: the_file
    datatype: file
---
code: |
  success_sending_email = send_email(to=[user], template=hello_email, attachments=[the_file])
---
template: hello_email
subject: |
  A picture for ${ user }
content: |
  Hello, ${ user }!

  Attached please find an incredibly *cool* picture.

  From,

  Your friend
---
{% endhighlight %}

## <a name="send_sms"></a>send_sms()

The `send_sms()` function is similar to `send_email()`, except it
sends a text message (also known as an [SMS] message).  This requires
a [Twilio] account.

All of its arguments are [keyword arguments], the defaults of which
are:

{% highlight python %}
send_sms(to=None, body=None, template=None, task=None, attachments=None, config='default')
{% endhighlight %}

This function is integrated with other classes in
[`docassemble.base.util`] and [`docassemble.base.core`].

* `to` expects a [list] of recipients.  The list can consist of
  [`Individual`]s (or any other [`Person`]s), objects of type
  [`phonenumbers.PhoneNumber`], or a simple string containing a phone number.
* `body` expects text, or `None`.  If provided, it will be the content
  of the message.  Markdown will be converted to plain text.
* `template` expects a [`DATemplate`] object, or `None`.  These
  templates can be created in an interview file using the `template`
  directive.  The "subject" of the template, if provided, will be the first
  line of the message.
* `task` expects the name of a [task].  If this argument is provided,
  and if sending the text message is successful, the task will be
  marked as having been performed (i.e., [`mark_task_as_performed()`]
  will be called).  Alternatively, you can handle this in your own
  code, but you might find it convenient to let the `send_email()`
  function handle it for you.
* `attachments` expects a [list] of [`DAFile`] objects, [`DAFileList`]
  objects, [`DAFileCollection`] objects, or ordinary URLs.  If
  provided, the message will be an [MMS] message containing the
  attached files.  No more than 10 attachments may be added.  You can
  include:
  * Images generated by `signature` blocks (objects of class
  [`DAFile`]);
  * File uploads generated by including [fields] of `datatype: file` or
  `datatype: files` in a [`question`] (objects of class [`DAFileList`]);
  * [Documents] generated by [`attachments`] to a [`question`] for which a
  `variable` was provided (objects of class [`DAFileCollection`]).

When the recipients are [`Individual`]s or [`Person`]s, the
`mobile_number` attribute will be used, but only if it already exists.
Otherwise, the `phone_number` attribute will be used, and sought if it
is not already defined.

Note that [Twilio] expects the phone number to be expressed in [E.164]
format, which includes the country code (e.g., 1 for the United
States).  However, users do not typically write phone numbers in such
a way.  Therefore, the [`phonenumbers`] package is used to convert
phone numbers to [E.164] based on the applicable country.  If an
[`Individual`] or [`Person`] is the recipient, the `country`
attribute, if it exists, will be used to determine the country.
Otherwise, the [`get_country()`] function is used to determine the
applicable country.  Your interview can use [`set_country()`] in
[`initial`] code to set a default country, or you can set a default on
a server level by setting the [`country` configuration directive].
The country must be specified as a two-letter, capitalized
abbreviation.

`send_sms()` returns `False` if an error prevented the message from
being sent; otherwise it returns `True`.

See the [`twilio` configuration directive] for information about how to configure that `send_sms()` will use.

## <a name="prevent_going_back"></a>prevent_going_back()

**docassemble**'s back button helps users when they make a mistake and
want to go back and correct it.  But sometimes, we want to prevent
users from going back.  For example, if the interview code causes an
e-mail to be sent, or data to be written to a database, allowing the
user to go back and do the process again would create confusion.

You can call `prevent_going_back()` to instruct the web application to
prevent the user from going back past that point.  See also the
[modifier] of the same name.

## <a name="selections"></a>selections()

This is used in multiple choice questions in `fields` lists where the
`datatype` is `object`, `object_radio`, or `object_list` and the list
of selections is created by embedded `code`.  The function takes one
or more arguments and outputs an appropriately formatted list of
objects.  If any of the arguments is a list, the list is unpacked and
its elements are added to the list of selections.

## <a name="objects_from_file"></a>objects_from_file()

`objects_from_file()` is a utility function for initializing a group
of objects from a [YAML] file written in a certain format.

# <a name="storage"></a>Storing data

## <a name="redis"></a>With Redis

If you do not know what a [Redis] server is, skip this section!

The [background processes] feature of **docassemble** depends on a
[Redis] server being available.  The server is also used to facilitate
[live chat].

Interview authors may want to make use of the [Redis] server for
purposes of storing information across users of a particular
interview, keeping usage statistics, or other purposes.

{% include side-by-side.html demo="redis" %}

To use the [Redis] server, use an [`objects`] section to create an
object of type `DARedis`.  This object can now be used to
communicate with the redis server, much as though it had been created
by calling `redis.StrictRedis()`.

## <a name="sql"></a>With SQL

Since [Redis] is an [in-memory database], it is not appropriate for
long-term storage or for the storage of large amounts of data.

An alternative is to store data in SQL.  **docassemble** provides three
functions that allow you to store, retrieve, and delete data.

{% include side-by-side.html demo="database_storage" %}

<a name="write_record"></a>When you call `write_record(key, data)`,
the variable `data` is stored in a SQL database.  The function returns
the integer unique ID for the record.  The `data` variable can be any
type of data, such as a number, some text, an [object], a
[Python dictionary], or something else.  The only limitation is that
the information in the variable needs to be able to be [pickled].

<a name="read_records"></a>When you call `read_records(key)`, you will
retrieve all of the records that had been stored using that `key`.
The function returns a [Python dictionary] where the keys are integers
representing the unique ID of the record.

<a name="delete_records"></a>You can delete records by calling
`delete_record(key, id)` where `key` is the key under which the record
was saved and `id` is the unique ID integer of the record.

Note that all interviews on the server will have access to the data
stored with `write_record()`, and the data are not encrypted on the
server.  It is important to choose your `key` names wisely because if
you use a simple name like `mydata`, another interview developer might
have chosen the same key, and then your data will become intermingled.
It is a good idea to include your interview package name in the `key`
names you choose.

**docassemble** will attempt to sanitize data you pass to
`write_record()` by converting any item that cannot be [pickled] to
`None`.

### Advanced SQL storage

Though the data are stored on a SQL server, you cannot use SQL queries
to retrieve data stored by `write_record()`; you can only use
`read_records()` to retrieve data.

If you want to use the full power of SQL in your interviews, you can
write a module that does something like this:

{% highlight python %}
from docassemble.base.util import get_config
import psycopg2

def get_conn():
    dbconfig = get_config('db')
    return psycopg2.connect(database=dbconfig.get('name'),
                            user=dbconfig.get('user'),
                            password=dbconfig.get('password')
                            host=dbconfig.get('host')
                            port=dbconfig.get('port'))
def some_function(id, thing):
    conn = get_conn()
    cur = conn.get_cursor()
    cur.execute("update foo set bar=%s where id=%s", (thing, id))
    conn.commit()
    cur.close()
{% endhighlight %}

This assumes the [`db`] configuration refers to a [PostgreSQL]
database.  If you connect to the database with the credentials from
[`db`], you have the power to create and drop tables.

# Writing your own functions

There are two ways that you can write your own functions in
**docassemble**.

The first way is to use the `<%def></%def>` feature of [Mako] in order
to use "functions" in your templates.  These are not true
[Python functions] because they are based around [Mako] templates, but
they are similar to [Python functions].  The `<%def></%def>` feature
is documented on the [Mako web site].  **docassemble**'s [`def` block]
makes it easy to re-use [`def` blocks] in your interviews.

The second way, which is usually more elegant, is to write a
[Python module] containing a definition of a [Python function], and
then include that module in your interview using the
[`modules` block].  This allows you to use your function both in
[Mako] templates and in [`code`] blocks.

Here is a brief tutorial on how to write a function `plus_one()` that
takes a number and returns the number plus one.  For example,
`plus_one(3)` should return `4`.

First, go to the [Playground].  (This requires that you have a
developer account on the server.)

Then, go to the [modules folder] of the [Playground].

![modules folder]({{ site.baseurl }}/img/docassemble-modules.png)

Then, type the following [Python] code into the text box:

{% highlight python %}
def plus_one(number):
  return number + 1
{% endhighlight %}

The screen should look like this:

![sample function]({{ site.baseurl }}/img/docassemble-sample-function.png)

Then, press the "Save" button at the bottom of the screen.  This will
create a [Python module] called `test`.  (The text file is called
`test.py` in the [modules folder] because files containing Python code
have the extension `.py` in their file names.  Within [Python], you
refer to modules using the file name without the file extension.)

Then click the "Back to Playground" button to leave the "Modules"
folder.

Now, you can use the `plus_one()` function in your interviews by
doing something like:

{% include side-by-side.html demo="sample-function" %}

The `.` in front of the module name is [Python]'s way of referring to
modules that are in the current package.  If you put your module
within a [package] called `docassemble.simplemath`, then you could
write, instead:

{% highlight yaml %}
---
modules:
  - docassemble.simplemath.test
---
{% endhighlight %}

[package]: {{ site.baseurl }}/docs/playground.html#packages
[modules folder]: {{ site.baseurl }}/docs/playground.html#modules
[Playground]: {{ site.baseurl }}/docs/playground.html
[Mako web site]: http://docs.makotemplates.org/en/latest/defs.html
[Mako]: http://www.makotemplates.org/
[Content-Type header]: https://en.wikipedia.org/wiki/List_of_HTTP_header_fields
[Documents]: {{ site.baseurl }}/docs/documents.html
[Flask-Mail]: https://pythonhosted.org/Flask-Mail/
[HTML]: https://en.wikipedia.org/wiki/HTML
[JSON]: https://en.wikipedia.org/wiki/JSON
[JSON]: https://en.wikipedia.org/wiki/JSON
[Markdown]: https://daringfireball.net/projects/markdown/
[Python dictionary]: https://docs.python.org/2/tutorial/datastructures.html#dictionaries
[Python function]: https://docs.python.org/2/tutorial/controlflow.html#defining-functions
[Python functions]: https://docs.python.org/2/tutorial/controlflow.html#defining-functions
[Python interpreter]: https://docs.python.org/2/tutorial/interpreter.html
[Python list]: https://docs.python.org/2/tutorial/datastructures.html
[Python locale]: https://docs.python.org/2/library/locale.html
[Python module]: https://docs.python.org/2/tutorial/modules.html
[Python modules]: https://docs.python.org/2/tutorial/modules.html
[Python set]: https://docs.python.org/2/library/stdtypes.html#set
[Python]: https://en.wikipedia.org/wiki/Python_%28programming_language%29
[Setting Variables]: {{ site.baseurl }}/docs/fields.html
[WSGI]: http://en.wikipedia.org/wiki/Web_Server_Gateway_Interface
[YAML]: https://en.wikipedia.org/wiki/YAML
[`Address`]: {{ site.baseurl }}/docs/objects.html#Address
[`DAFileCollection`]: {{ site.baseurl }}/docs/objects.html#DAFileCollection
[`DAFileList`]: {{ site.baseurl }}/docs/objects.html#DAFileList
[`DAFile`]: {{ site.baseurl }}/docs/objects.html#DAFile
[`DATemplate`]: {{ site.baseurl }}/docs/objects.html#DATemplate
[`Individual`]: {{ site.baseurl }}/docs/objects.html#Individual
[`LatitudeLongitude`]: {{ site.baseurl }}/docs/objects.html#LatitudeLongitude
[`Organization`]: {{ site.baseurl }}/docs/objects.html#Organization
[`Person`]: {{ site.baseurl }}/docs/objects.html#Person
[`attachments`]: {{ site.baseurl }}/docs/documents.html#attachments
[`basic-questions.yml`]: {{ site.github.repository_url }}/blob/master/docassemble_base/docassemble/base/data/questions/basic-questions.yml
[`code`]: {{ site.baseurl }}/docs/code.html
[`currency()`]: #currency
[`currency_symbol()`]: #currency_symbol
[`currency_symbol`]: {{ site.baseurl }}/docs/config.html#currency_symbol
[`default role`]: {{ site.baseurl }}/docs/initial.html#default role
[`docassemble.base.core`]: {{ site.github.repository_url }}/blob/master/docassemble_base/docassemble/base/core.py
[`docassemble.base.legal`]: {{ site.github.repository_url }}/blob/master/docassemble_base/docassemble/base/legal.py
[`docassemble.base.util`]: {{ site.github.repository_url }}/blob/master/docassemble_base/docassemble/base/util.py
[`docassemble.base`]: {{ site.baseurl }}/docs/installation.html#docassemble.base
[`event`]: {{ site.baseurl }}/docs/fields.html#event
[`force_ask()`]: #force_ask
[`force_gather()`]: #force_gather
[`get_info()`]: #get_info
[`initial`]: {{ site.baseurl }}/docs/logic.html#initial
[`interview_url()`]: #interview_url
[`interview_url_action()`]: #interview_url_action
[`legal.py`]: {{ site.github.repository_url }}/blob/master/docassemble_base/docassemble/base/legal.py
[`mandatory`]: {{ site.baseurl }}/docs/logic.html#mandatory
[`modules`]: {{ site.baseurl }}/docs/initial.html#modules
[`modules` block]: {{ site.baseurl }}/docs/initial.html#modules
[`multi_user`]: {{ site.baseurl }}/docs/special.html#multi_user
[`need()`]: #need
[`nice_number()`]: #nice_number
[`noun_plural()`]: #noun_plural
[`object_possessive`]: {{ site.baseurl }}/docs/objects.html#DAObject.object_possessive
[`objects`]: {{ site.baseurl }}/docs/initial.html#objects
[`process_action()`]: #process_action
[`question`]: {{ site.baseurl }}/docs/questions.html#question
[`set_info()`]: #set_info
[`set_locale()`]: #set_locale
[`subquestion`]: {{ site.baseurl }}/docs/questions.html#subquestion
[`title_case()`]: #title_case
[`track_location`]:  {{ site.baseurl }}/docs/special.html#track_location
[`url_action()`]: #url_action
[`words`]: {{ site.baseurl }}/docs/config.html#words
[action]: #url_action
[`attachment`]: {{ site.baseurl }}/docs/documents.html
[attachments]: {{ site.baseurl }}/docs/documents.html
[classes]: https://docs.python.org/2/tutorial/classes.html
[configuration]: {{ site.baseurl }}/docs/config.html
[dictionary]: https://docs.python.org/2/tutorial/datastructures.html#dictionaries
[fields]: {{ site.baseurl }}/docs/fields.html
[function]: https://docs.python.org/2/tutorial/controlflow.html#defining-functions
[functions]: #functions
[how **docassemble** runs your code]: {{ site.baseurl }}/docs/logic.html#howitworks
[html2text]: https://pypi.python.org/pypi/html2text
[initial block]: {{ site.baseurl }}/docs/initial.html
[initial blocks]: {{ site.baseurl }}/docs/initial.html
[installation]: {{ site.baseurl }}/docs/installation.html
[interview logic]: {{ site.baseurl }}/docs/logic.html
[keyword argument]: https://docs.python.org/2/glossary.html#term-argument
[keyword arguments]: https://docs.python.org/2/glossary.html#term-argument
[language support]: {{ site.baseurl }}/docs/language.html
[list]: https://docs.python.org/2/tutorial/datastructures.html
[locale module]: https://docs.python.org/2/library/locale.html
[markup]: {{ site.baseurl }}/docs/markup.html
[methods]: {{ site.baseurl }}/docs/objects.html#person classes
[modifier]: {{ site.baseurl }}/docs/modifiers.html#prevent_going_back
[multi-user interviews]: {{ site.baseurl }}/docs/roles.html
[objects]: {{ site.baseurl }}/docs/objects.html
[pattern.en]: http://www.clips.ua.ac.be/pages/pattern-en
[question]: {{ site.baseurl }}/docs/questions.html
[roles]: {{ site.baseurl }}/docs/roles.html
[scheduled task]: {{ site.baseurl }}/docs/scheduled.html
[scheduled tasks]: {{ site.baseurl }}/docs/scheduled.html
[special variable]: {{ site.baseurl }}/docs/special.html
[special variables]: {{ site.baseurl }}/docs/special.html
[titlecase]: https://pypi.python.org/pypi/titlecase
[user login system]: {{ site.baseurl }}/docs/users.html
[cron user]: {{ site.baseurl }}/docs/scheduled.html#cron user
[`last_access_time()`]: #last_access_time
[`last_access_days()`]: #last_access_days
[`last_access_hours()`]: #last_access_hours
[`last_access_minutes()`]: #last_access_minutes
[`last_access_delta()`]: #last_access_delta
[UTC time]: https://en.wikipedia.org/wiki/Coordinated_Universal_Time
[`timezone_list()`]: #timezone_list
[`pytz`]: http://pytz.sourceforge.net/
[`timezone` configuration]: {{ site.baseurl }}/docs/config.html#timezone
[`datetime.timedelta`]: https://docs.python.org/2/library/datetime.html#datetime.timedelta
[`dateutil.relativedelta.relativedelta`]: http://dateutil.readthedocs.io/en/stable/relativedelta.html
[`datetime`]: https://docs.python.org/2/library/datetime.html#datetime.datetime
[`action_argument()`]: #action_argument
[`action_arguments()`]: #action_arguments
[HTTPS]: {{ site.baseurl }}/docs/docker.html#https
[QR code]: https://en.wikipedia.org/wiki/QR_code
[Profile page]: {{ site.baseurl }}/docs/users.html#profile
[country code]: http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2
[`task_performed()`]: {{ site.baseurl }}/docs/functions.html#task_performed
[`task_not_yet_performed()`]: #task_not_yet_performed
[`mark_task_as_performed()`]: #mark_task_as_performed
[`times_task_performed()`]: #times_task_performed
[`set_task_counter()`]: #set_task_counter
[task]: #tasks
[actions]: #actions
[`format_date()`]: #format_date
[`format_time()`]: #format_time
[`user_info()`]: #user_info
[`user_logged_in()`]: #user_logged_in
[`user_has_privilege()`]: #user_logged_in
[`def` block]: {{ site.baseurl }}/docs/initial.html#def
[`def` blocks]: {{ site.baseurl }}/docs/initial.html#def
[`response()`]: #response
[`command()`]: #command
[`message()`]: #message
[Python Imaging Library]: http://www.pythonware.com/products/pil/
[URL arguments]: {{ site.baseurl }}/docs/special.html#url_args
[Celery]: http://www.celeryproject.org/
[`background_action()`]: #background_action
[`background_response()`]: #background_response
[`background_response_action()`]: #background_response_action
[callback function]: https://en.wikipedia.org/wiki/Callback_(computer_programming)
[background processes]: #background
[background process]: #background
[live chat]: {{ site.baseurl }}/docs/livehelp.html
[special variable `speak_text`]: {{ site.baseurl }}/docs/special.html#speak_text
[`country` configuration setting]: {{ site.baseurl }}/docs/config.html#country
[SMS]: https://en.wikipedia.org/wiki/Short_Message_Service
[MMS]: https://en.wikipedia.org/wiki/Multimedia_Messaging_Service
[Twilio]: https://twilio.com
[E.164]: https://support.twilio.com/hc/en-us/articles/223183008-Formatting-International-Phone-Numbers
[`phonenumbers.PhoneNumber`]: https://github.com/daviddrysdale/python-phonenumbers
[`twilio` configuration directive]: {{ site.baseurl }}/docs/config.html#twilio
[`phonenumbers`]: https://github.com/daviddrysdale/python-phonenumbers
[`get_country()`]: #get_country
[`set_country()`]: #set_country
[`country` configuration directive]: {{ site.baseurl }}/docs/config.html#country
[`pycountry` package]: https://pypi.python.org/pypi/pycountry
[`send_sms()`]: #send_sms
[flash]: http://flask.pocoo.org/docs/0.11/patterns/flashing/
[Javascript]: https://en.wikipedia.org/wiki/JavaScript
[`alert()`]: http://www.w3schools.com/jsref/met_win_alert.asp
[multiple servers]: {{ site.baseurl }}/docs/scalability.html
[special user]: {{ site.baseurl }}/docs/scheduled.html#cron user
[Redis]: https://redis.io/
[in-memory database]: https://en.wikipedia.org/wiki/In-memory_database
[object]: {{ site.baseurl }}/docs/objects.html
[pickled]: https://docs.python.org/2/library/pickle.html
[`db`]: {{ site.baseurl }}/docs/config.html#db
[PostgreSQL]: http://www.postgresql.org/
[`checkin interval`]: {{ site.baseurl }}/docs/config.html#checkin interval