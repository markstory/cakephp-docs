CakeEmail
#########

.. php:class:: CakeEmail

``CakeEmail`` is a new class to send email. With this
class you can send email from any place of your application. In addition to
using the EmailComponent from your controller, you can also send mail from
Shells, and Models.

This class replaces the :php:class:`EmailComponent` and gives more flexibility
in sending emails. For example, you can create your own transports to send
email instead of using the provided smtp and mail.

Basic usage
===========

First of all, you should ensure the class is loaded using :php:meth:`App::uses()`::

    <?php
    App::uses('CakeEmail', 'Network/Email');

Using CakeEmail is similar to using :php:class:`EmailComponent`. But instead of
using attributes you must use methods. Example::

    <?php
    $email = new CakeEmail();
    $email->from('me@example.com');
    $email->to('you@example.com');
    $email->subject('About');
    $email->send('My message');

To simplify things, all of the setter methods return the instance of class. You can re-write the
above code as::

    <?php
    $email = new CakeEmail();
    $email->from('me@example.com')
        ->to('you@example.com')
        ->subject('About')
        ->send('My message');

Choosing the sender
-------------------

When sending email on behalf of other people its often a good idea to define the
original sender using the Sender header.  You can do so using ``sender()``::

    <?php
    $email = new CakeEmail();
    $email->sender('app@example.com', 'MyApp emailer');

.. note::

    Its also a good idea to set the envelope sender when sending mail on another
    person's behalf.  This prevents them from getting any messages about
    deliverability.

Configuration
-------------

Similar of database configuration, emails also have a class to centralize all the
configuration.

You should create the file ``app/Config/email.php`` with the class
``EmailConfig``. The ``app/Config/email.php.default`` has an example of this
file.

The ``CakeEmail`` create an instance of this file before use the config. If you
have a variable data to put in the configs, you can use the constructor to do
that.

You also can configure SMTP servers with SSL connection, like GMail. To it, put
``'ssl://'`` at prefix in the host and configure properly the port value.
Example::

    public $gmail = array(
        'host' => 'ssl://smtp.gmail.com',
        'port' => 465,
        'username' => 'my@gmail.com',
        'password' => 'secret'
    );

.. note::

    To have this feature you need to have the SSL configured in your PHP
    install.

Setting headers
---------------

In ``EmailComponent`` you only can set the ``X-*`` headers. In ``CakeEmail`` you
are free to set the headers you want. When migrating to use CakeEmail, do not
forget to put the ``X-`` prefix in your headers.

Email templates
---------------

The paths are changed from elements path to views path directly. So, the email
templates are now located in ``app/View/Emails/html/template.ctp`` and
``app/View/Emails/text/template.ctp``. The layouts are changed to
``app/View/Layout/Emails``.

Sending attachments
-------------------

The attachment format has changed. Below are the new forms:

1. String: ``$email->attachments('/full/file/path/file.png')`` will attach this
   file with the name file.png.
2. Array: ``$email->attachments(array('/full/file/path/file.png')`` will have
   the same behavior as using a string.
3. Array with key:
   ``$email->attachments(array('photo.png' => '/full/some_hash.png'))`` will
   attach some_hash.png with the name photo.png. The recipient will see
   photo.png, not some_hash.png.
4. Array with arrays::
   
        <?php
        $email->attachments(array(
            'photo.png' => array(
                'file' => '/full/some_hash.png',
                'mimetype' => 'image/png',
                'contentId' => 'my-unique-id'
            )
        ));

   The above will attach the file with different mimetype and with custom Content ID
   (when set the content ID the attachment is transformed to inline). The
   mimetype and contentId are optional in this form.

  4.1. When you are using the ``contentId``, you can use the file in the html
  body like ``<img src="cid:my-content-id">``.

Using transports
----------------

Transports are classes designed to send the e-mail over some protocol or method.
CakePHP support the Mail (default) and Smtp transports.

To configure your method, you must use the :php:meth:`CakeEmail::transport()` method.

Creating custom Transports
~~~~~~~~~~~~~~~~~~~~~~~~~~

You are able to create your custom transports to integrate with others emails
systems (like SwiftMailer). To it, you must to create the file
``app/Network/Email/ExampleTransport.php`` (where Example is the name of your
transport).

This file must contain the class ``ExampleTransport`` extending
``AbstractTransport`` (do not forget to use
``App::uses('AbstractTransport', 'Network/Email');`` before).

You must implement the method ``send(CakeEmail $email)`` with your custom logic.
Optionally, you can implement the method ``config($config)`` that is called
before the send to pass the user configurations. By default, this method put the
configuration in protected attribute ``$_config``.

If you need to call some method from this transport before send, you can call
:php:meth:`CakeEmail::transportClass()` to get an instance of
transport. Example::

    <?php
    $yourInstance = $email->transport('your')->transportClass();
    $yourInstance->myCustomMethod();
    $email->send();


Sending messages quickly
========================

Sometimes you need a quick way to fire off an email, and you don't necessarily
want do setup a bunch of configuration ahead of time. 
:php:meth:`~CakeEmail::deliver()` is intended for that purpose.

You can create a configuration in ``EmailConfig`` or an array with all options
that you need and use the static method ``CakeEmail::deliver()``. Example::

    <?php
    CakeEmail::deliver('you@example.com', 'Subject', 'Message', array('from' => 'me@example.com'));

This method will send an email to you@example.com, from me@example.com with
subject Subject and content Message.

The return is a :php:class:`CakeEmail` instance with all configurations setted.
If you do not want send the email and configure something more before send, you
can pass the 5th parameter as false.

The 3rd parameter is the content of message or an array with variables (when
using rendered content).

The 4th parameter can be an array with the configurations or a string with the
name of configuration in ``EmailConfig``.

If you want, you can pass the to, subject and message as null and do all
configurations in the 4th parameter (as array or using ``EmailConfig``). The
follow configurations are used:

-  ``'from'``: Email or array of sender. See ``CakeEmail::from()``.
-  ``'sender'``: Email or array of real sender. See ``CakeEmail::sender()``.
-  ``'to'``: Email or array of destination. See ``CakeEmail::to()``.
-  ``'cc'``: Email or array of carbon copy. See ``CakeEmail::cc()``.
-  ``'bcc'``: Email or array of blind carbon copy. See ``CakeEmail::bcc()``.
-  ``'replyTo'``: Email or array to reply the e-mail. See ``CakeEmail::replyTo()``.
-  ``'readReceipt'``: Email or array to receive the receipt of read. See ``CakeEmail::readReceipt()``.
-  ``'returnPath'``: Email or array to return if have some error. See ``CakeEmail::returnPath()``.
-  ``'messageId'``: Message ID of e-mail. See ``CakeEmail::messageId()``.
-  ``'subject'``: Subject of the message. See ``CakeEmail::subject()``.
-  ``'message'``: Content of message. Do not set this field if you are using rendered content.
-  ``'headers'``: Headers to be included. See ``CakeEmail::setHeaders()``.
-  ``'viewRender'``: If you are using rendered content, set the view classname. See ``CakeEmail::viewRender()``.
-  ``'template'``: If you are using rendered content, set the template name. See ``CakeEmail::template()``.
-  ``'layout'``: If you are using rendered content, set the layout to render. If you want to render a template without layout, set this field to null. See ``CakeEmail::template()``.
-  ``'viewVars'``: If you are using rendered content, set the array with variables to be used in the view. See ``CakeEmail::viewVars()``.
-  ``'attachments'``: List of files to attach. See ``CakeEmail::attachments()``.
-  ``'emailFormat'``: Format of email (html, text or both). See ``CakeEmail::emailFormat()``.
-  ``'transport'``: Transport name. See ``CakeEmail::transport()``.

All these configurations are optional, except ``'from'``. If you put more
configurations in this array, these configurations will be used in the
:php:meth:`CakeEmail::config()` method. For example, if you are using smtp transport,
you should pass the host, port and others configurations.

.. todo::

    More information on the various method CakeEmail provides
    and how to use them all.  There is a good start here, but
    more detail would be good.
