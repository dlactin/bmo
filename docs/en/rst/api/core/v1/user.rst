Users
=====

This part of the Bugzilla API allows you to create user accounts, get information
about user accounts and to log in or out using an existing account.

.. _rest_user_login:

Login
-----

Logging in with a username and password is required for many Bugzilla
installations, in order to search for private bugs, post new bugs, etc. This
method allows you to retrieve a token that can be used as authentication for
subsequent API calls. Otherwise you will need to pass your ``login`` and
``password`` with each call.

This method will be going away in the future in favor of using *API keys*.

**Request**

.. code-block:: text

   GET /rest/login?login=foo@example.com&password=toosecrettoshow

==============  =======  ========================================================
name            type     description
==============  =======  ========================================================
**login**       string   The user's login name.
**password**    string   The user's password.
==============  =======  ========================================================

**Response**

.. code-block:: js

   {
     "token": "786-OLaWfBisMY",
     "id": 786
   }

========  ======  ===============================================================
name      type    description
========  ======  ===============================================================
id        int     Numeric ID of the user that was logged in.
token     string  Token which can be passed in the parameters as
                  authentication in other calls. The token can be sent along
                  with any future requests to the webservice, for the duration
                  of the session, i.e. til :ref:`rest_user_logout` is called.
========  ======  ===============================================================

**Errors**

* 300 (Invalid Username or Password)
  The username does not exist, or the password is wrong.
* 301 (Login Disabled)
  The ability to login with this account has been disabled.  A reason may be
  specified with the error.
* 305 (New Password Required)
  The current password is correct, but the user is asked to change
  their password.
* 50 (Param Required)
  A login or password parameter was not provided.

.. _rest_user_logout:

Logout
------

Log out the user. Basically it invalidates the token provided so it cannot be
re-used. Does nothing if the token is not in use. Will also clear any existing
authentication cookies the client may still have stored.

**Request**

.. code-block:: text

   GET /rest/logout?token=1234-VWvO51X69r

=====  ======  ===================================================
name   type    description
=====  ======  ===================================================
token  string   The user's token used for authentication.
=====  ======  ===================================================

.. _rest_user_valid_login:

Valid Login
-----------

This method will verify whether a client's cookies or current login token is
still valid or have expired. A valid username that matches must be provided as
well.

**Request**

.. code-block:: text

   GET /rest/valid_login?login=foo@example.com&token=1234-VWvO51X69r

=========  =======  =============================================================
name       type     description
=========  =======  =============================================================
**login**  string   The login name that matches the provided cookies or token.
token      string   Persistent login token currently being used for
                    authentication.
=========  =======  =============================================================

**Response**

Returns true/false depending on if the current token is valid for the provided
username.

.. _rest_user_create:

Create User
-----------

Creates a user account directly in Bugzilla, password and all. Instead of this,
you should use **Offer Account by Email** when possible because that makes sure
that the email address specified can actually receive an email. This function
does not check that. You must be authenticated and be in the *editusers* group
to perform this action.

**Request**

.. code-block:: text

   POST /rest/user

.. code-block:: js

   {
     "email" : "user@bugzilla.org",
     "full_name" : "Test User",
     "password" : "K16ldRr922I1"
   }

============  ======  =============================================================
name          type    description
============  ======  =============================================================
**email**     string  The email address for the new user.
full_name     string  The user's full name. Will be set to empty if not specified.
password      string  The password for the new user account, in plain text. It
                      will be stripped of leading and trailing whitespace. If
                      blank or not specified, the new created account will
                      exist in Bugzilla but will not be allowed to log in
                      using DB authentication until a password is set either
                      by the user (through resetting their password) or by the
                      administrator.
iam_username  string  (optional) The IAM username used to authenticate with using
                      an external IAM system
============  ======  =============================================================

**Response**

.. code-block:: js

   {
     "id": 58707
   }

====  ====  ============================================
name  type  description
====  ====  ============================================
id    int   The numeric ID of the user that was created.
====  ====  ============================================

**Errors**

* 502 (Password Too Short)
  The password specified is too short. (Usually, this means the
  password is under three characters.)

.. _rest_user_update:

Update User
-----------

Updates an existing user account in Bugzilla. You must be authenticated and be
in the *editusers* group to perform this action.

**Request**

.. code-block:: text

   PUT /rest/user/(id_or_name)

You can edit a single user by passing the ID or login name of the user
in the URL. To edit more than one user, you can specify addition IDs or
login names using the ``ids`` or ``names`` parameters respectively.

=================  =======  =====================================================
 name              type     description
=================  =======  =====================================================
**id_or_name**     mixed    Either the ID or the login name of the user to
                            update.
**ids**            array    Additional IDs of users to update.
**names**          array    Additional login names of users to update.
full_name          string   The new name of the user.
email              string   The email of the user. Note that email used to
                            login to Bugzilla. Also note that you can only
                            update one user at a time when changing the login
                            name / email. (An error will be thrown if you try to
                            update this field for multiple users at once.)
password           string   The password of the user.
email_enabled      boolean  A boolean value to enable/disable sending
                            bug-related mail to the user.
login_denied_text  string   A text field that holds the reason for disabling a
                            user from logging into Bugzilla. If empty, then the
                            user account is enabled; otherwise it is
                            disabled/closed.
groups             object   These specify the groups that this user is directly
                            a member of. To set these, you should pass an object
                            as the value. The object's items are described in
                            the Groups update objects below.
bless_groups       object   This is the same as groups but affects what groups
                            a user has direct membership to bless that group.
                            It takes the same inputs as groups.
iam_username       string   (optional) The IAM username used to authenticate with using
                            an external IAM system
=================  =======  =====================================================

Groups and bless groups update object:

======  =====  ==================================================================
name    type   description
======  =====  ==================================================================
add     array  The group IDs or group names that the user should be added to.
remove  array  The group IDs or group names that the user should be removed from.
set     array  Integers or strings which are an exact set of group IDs and group
               names that the user should be a member of. This does not remove
               groups from the user when the person making the change does not
               have the bless privilege for the group.
======  =====  ==================================================================

If you specify ``set``, then ``add`` and ``remove`` will be ignored. A group in
both the ``add`` and ``remove`` list will be added. Specifying a group that the
user making the change does not have bless rights will generate an error.

**Response**

* users: (array) List of user change objects with the following items:

=======  ======  ================================================================
name     type    description
=======  ======  ================================================================
id       int     The ID of the user that was updated.
changes  object  The changes that were actually done on this user. The keys
                 are the names of the fields that were changed, and the values
                 are an object with two items:

                 * added: (string) The values that were added to this field,
                   possibly a comma-and-space-separated list if multiple values
                   were added.
                 * removed: (string) The values that were removed from this
                   field, possibly a comma-and-space-separated list if multiple
                   values were removed.
=======  ======  ================================================================

**Errors**

* 51 (Bad Login Name)
  You passed an invalid login name in the "names" array.
* 304 (Authorization Required)
  Logged-in users are not authorized to edit other users.

.. _rest_user_get:

Get User
--------

Gets information about user accounts in Bugzilla.

**Request**

To get information about a single user in Bugzilla:

.. code-block:: text

   GET /rest/user/(id_or_name)

To get multiple users by name or ID:

.. code-block:: text

   GET /rest/user?names=foo@bar.com&names=test@bugzilla.org
   GET /rest/user?ids=123&ids=321

To get user matching a search string:

.. code-block:: text

   GET /rest/user?match=foo

To get user by using an integer ID value or by using ``match``, you must be
authenticated.

================  =======  ======================================================
name              type     description
================  =======  ======================================================
id_or_name        mixed    An integer user ID or login name of the user.
ids               array    Integer user IDs. Logged=out users cannot pass
                           this parameter to this function. If they try,
                           they will get an error. Logged=in users will get
                           an error if they specify the ID of a user they
                           cannot see.
names             array    Login names.
match             array    This works just like "user matching" in Bugzilla
                           itself. Users will be returned whose real name
                           or login name contains any one of the specified
                           strings. Users that you cannot see will not be
                           included in the returned list.

                           Most installations have a limit on how many
                           matches are returned for each string; the default
                           is 1000 but can be changed by the Bugzilla
                           administrator.

                           Logged-out users cannot use this argument, and
                           an error will be thrown if they try. (This is to
                           make it harder for spammers to harvest email
                           addresses from Bugzilla, and also to enforce the
                           user visibility restrictions that are
                           implemented on some Bugzillas.)
limit             int      Limit the number of users matched by the
                           ``match`` parameter. If the value is greater than the
                           system limit, the system limit will be used.
                           This parameter is only valid when using the ``match``
                           parameter.
group_ids         array    Numeric IDs for groups that a user can be in.
groups            array    Names of groups that a user can be in. If
                           ``group_ids`` or ``groups`` are specified, they
                           limit the return value to users who are in *any*
                           of the groups specified.
include_disabled  boolean  By default, when using the ``match`` parameter,
                           disabled users are excluded from the returned
                           results unless their full username is identical
                           to the match string. Setting ``include_disabled`` to
                           ``true`` will include disabled users in the returned
                           results even if their username doesn't fully match
                           the input string.
permissive        boolean  When querying for users using names, do not fail the
                           entire request if one or more errors occur. A `faults`
                           list is included that contains the individual errors.
================  =======  ======================================================

**Response**

* users: (array) Each object describes a user and has the following items:

==================  ========  =====================================================
name                type      description
==================  ========  =====================================================
id                  int       The unique integer ID that Bugzilla uses to represent
                              this user. Even if the user's login name changes,
                              this will not change.
real_name           string    The actual name of the user. May be blank.
nick                string    The user's nickname. Currently this is extracted from
                              the real_name, name or email field.
email               string    The email address of the user.
name                string    The login name of the user. Note that in some
                              situations this is different than their email.
can_login           boolean   A boolean value to indicate if the user can login
                              into Bugzilla.
email_enabled       boolean   A boolean value to indicate if bug-related mail will
                              be sent to the user or not. Only users in the
                              *disableusers* group can see this field.
login_denied_text   string    A text field that holds the reason for disabling a
                              user from logging into Bugzilla. If empty then the
                              user account is enabled; otherwise it is
                              disabled/closed. Only users in the *disableusers*
                              group can see this field.
groups              array     Groups the user is a member of. If the currently
                              logged in user is querying their own account or is a
                              member of a privileged permission group, the array will
                              contain all the groups that the user is a member of.
                              Otherwise, the array will only contain groups that
                              the logged in user can bless. Each object describes
                              the group and contains the items described in the
                              Group object below.
saved_searches      array     User's saved searches, each having the following
                              Search object items described below.
saved_reports       array     User's saved reports, each having the following
                              Search object items described below.
last_seen_date      datetime  The time when the user last loaded any page.
last_activity_time  datetime  The time when the user last made a change to a bug.
==================  ========  =====================================================

Group object:

===========  ======  ============================================================
name         type    description
===========  ======  ============================================================
id           int     The group ID
name         string  The name of the group
description  string  The description for the group
===========  ======  ============================================================

Search object:

=====  ======  ==================================================================
name   type    description
=====  ======  ==================================================================
id     int     An integer ID uniquely identifying the saved report.
name   string  The name of the saved report.
query  string  The CGI parameters for the saved report.
=====  ======  ==================================================================

If you are not authenticated when you call this function, you will only be
returned the ``id``, ``name``, ``real_name`` and ``nick`` items. If you are
authenticated and not in 'editusers' group, you will only be returned the ``id``,
``name``, ``real_name``, ``nick``, ``email``, ``can_login`` and ``groups`` items.
The groups returned are filtered based on your permission to bless each group.
The ``saved_searches`` and ``saved_reports`` items are only returned if you are
querying your own account, even if you are in the editusers group.

**Errors**

* 51 (Bad Login Name or Group ID)
  You passed an invalid login name in the "names" array or a bad
  group ID in the "group_ids" argument.
* 52 (Invalid Parameter)
  The value used must be an integer greater than zero.
* 304 (Authorization Required)
  You are logged in, but you are not authorized to see one of the users you
  wanted to get information about by user id.
* 505 (User Access By Id or User-Matching Denied)
  Logged-out users cannot use the "ids" or "match" arguments to this
  function.
* 804 (Invalid Group Name)
  You passed a group name in the "groups" argument which either does not
  exist or you do not belong to it.

.. _rest_user_whoami:

Who Am I
--------

Allows for validating a user's API key, token, or username and password.
If successfully authenticated, it returns simple information about the
logged in user.

**Request**

.. code-block:: text

   GET /rest/whoami

**Response**

.. code-block:: js

   {
     "id" : "1234",
     "name" : "user@bugzilla.org",
     "real_name" : "Test User",
     "nick" : "user"
   }

========== ======  =====================================================
name       type    description
========== ======  =====================================================
id         int     The unique integer ID that Bugzilla uses to represent
                   this user. Even if the user's login name changes,
                   this will not change.
real_name  string  The actual name of the user. May be blank.
nick       string  The user's nickname. Currently this is extracted from
                   the real_name, name or email field.
name       string  string  The login name of the user.
========== ======  =====================================================
