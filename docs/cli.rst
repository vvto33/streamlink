Command-Line Interface
======================

Tutorial
--------

Streamlink is a command-line application, which means that the commands described
here should be typed into a terminal. On Windows, you have to open either the
`Command Prompt`_, `PowerShell`_ or `Windows Terminal`_, on macOS open the `Terminal <macOS-Terminal>`_ app,
and if you're on Linux or BSD you probably already know the drill.

The way Streamlink works is that it's only a means to extract and transport
the streams, and the playback is done by an external video player. Streamlink
works best with `VLC`_ or `mpv`_, which are also cross-platform, but other players
may be compatible too, see the :ref:`Players <players:Players>` page for a complete overview.

Now to get into actually using Streamlink, let's say you want to watch the
stream located on twitch.tv/day9tv, you start off by telling Streamlink
where to attempt to extract streams from. This is done by giving the URL to the
command :command:`streamlink` as the first argument:

.. code-block:: console

    $ streamlink twitch.tv/day9tv
    [cli][info] Found matching plugin twitch for URL twitch.tv/day9tv
    Available streams: audio, high, low, medium, mobile (worst), source (best)


.. note::
    You don't need to include the protocol when dealing with HTTP(s) URLs,
    e.g. just ``twitch.tv/day9tv`` is enough and quicker to type.


This command will tell Streamlink to attempt to extract streams from the URL
specified, and if it's successful, print out a list of available streams to choose
from.

In some cases  (`Supported streaming protocols`_)  local files are supported
using the ``file://`` protocol, for example a local HLS playlist can be played.
Relative file paths and absolute paths are supported. All path separators are ``/``,
even on Windows.

.. code-block:: console

    $ streamlink hls://file://C:/hls/playlist.m3u8
    [cli][info] Found matching plugin stream for URL hls://file://C:/hls/playlist.m3u8
    Available streams: 180p (worst), 272p, 408p, 554p, 818p, 1744p (best)


To select a stream and start playback, simply add the stream name as a second
argument to the :command:`streamlink` command:

.. sourcecode:: console

    $ streamlink twitch.tv/day9tv 1080p60
    [cli][info] Found matching plugin twitch for URL twitch.tv/day9tv
    [cli][info] Opening stream: 1080p60 (hls)
    [cli][info] Starting player: vlc


The stream you chose should now be playing in the player. It's a common use case
to just want to start the highest quality stream and not be bothered with what it's
named. To do this, just specify ``best`` as the stream name and Streamlink will
attempt to rank the streams and open the one of highest quality. You can also
specify ``worst`` to get the lowest quality.

Now that you have a basic grasp of how Streamlink works, you may want to look
into customizing it to your own needs, such as:

- Creating a :ref:`configuration file <cli:Configuration file>` of options you
  want to use
- Setting up your player to :ref:`cache some data <issues:Streams are buffering/lagging>`
  before playing the stream to help avoiding buffering issues


.. _Command Prompt: https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands
.. _PowerShell: https://docs.microsoft.com/en-us/powershell/
.. _Windows Terminal: https://docs.microsoft.com/en-us/windows/terminal/get-started
.. _macOS Terminal: https://support.apple.com/guide/terminal/welcome/mac
.. _VLC: https://videolan.org/
.. _mpv: https://mpv.io/


Configuration file
------------------

Writing the command-line options every time is inconvenient, that's why Streamlink
is capable of reading options from a configuration file instead.

Streamlink will look for config files in different locations depending on
your platform:

.. rst-class:: table-custom-layout table-custom-layout-platform-locations

================= ====================================================
Platform          Location
================= ====================================================
Linux, BSD        - ``${XDG_CONFIG_HOME:-${HOME}/.config}/streamlink/config``

                  Deprecated:

                  - ``${HOME}/.streamlinkrc``
macOS             - ``${HOME}/Library/Application Support/streamlink/config``

                  Deprecated:

                  - ``${XDG_CONFIG_HOME:-${HOME}/.config}/streamlink/config``
                  - ``${HOME}/.streamlinkrc``
Windows           - ``%APPDATA%\streamlink\config``

                  Deprecated:

                  - ``%APPDATA%\streamlink\streamlinkrc``
================= ====================================================

You can also specify the location yourself using the :option:`--config` option.

.. warning::

  On Windows, there is a default config created by the installer, but on any
  other platform you must create the file yourself.


Syntax
^^^^^^

The config file is a simple text file and should contain one
:ref:`command-line option <cli:Command-line usage>` (omitting the dashes) per
line in the format::

  option=value

or for an option without value::

  option

.. note::
    Any quotes used will be part of the value, so only use them when the value needs them,
    e.g. when specifying a player with a path which contains spaces.

Example
^^^^^^^

.. code-block:: bash

    # Player options
    player=mpv --cache 2048
    player-no-close

.. note::
    Full player paths are supported via configuration file options such as
    ``player="C:\mpv-x86_64\mpv"``


Plugin specific configuration file
----------------------------------

You may want to use specific options for some plugins only. This
can be accomplished by placing those settings inside a plugin specific
config file. Options inside these config files will override the main
config file when a URL matching the plugin is used.

Streamlink expects this config to be named like the main config but
with ``.<plugin name>`` attached to the end.

Examples
^^^^^^^^

.. rst-class:: table-custom-layout table-custom-layout-platform-locations

================= ====================================================
Platform          Location
================= ====================================================
Linux, BSD        - ``${XDG_CONFIG_HOME:-${HOME}/.config}/streamlink/config.pluginname``

                  Deprecated:

                  - ``${HOME}/.streamlinkrc.pluginname``
macOS             - ``${HOME}/Library/Application Support/streamlink/config.pluginname``

                  Deprecated:

                  - ``${XDG_CONFIG_HOME:-${HOME}/.config}/streamlink/config.pluginname``
                  - ``${HOME}/.streamlinkrc.pluginname``
Windows           - ``%APPDATA%\streamlink\config.pluginname``

                  Deprecated:

                  - ``%APPDATA%\streamlink\streamlinkrc.pluginname``
================= ====================================================

Have a look at the :ref:`list of plugins <plugin_matrix:Plugins>`, or
check the :option:`--plugins` option to see the name of each built-in plugin.


Sideloading plugins
-------------------

Streamlink will attempt to load standalone plugins from these directories:

.. rst-class:: table-custom-layout table-custom-layout-platform-locations

================= ====================================================
Platform          Location
================= ====================================================
Linux, BSD        - ``${XDG_DATA_HOME:-${HOME}/.local/share}/streamlink/plugins``

                  Deprecated:

                  - ``${XDG_CONFIG_HOME:-${HOME}/.config}/streamlink/plugins``
macOS             - ``${HOME}/Library/Application Support/streamlink/plugins``

                  Deprecated:

                  - ``${XDG_CONFIG_HOME:-${HOME}/.config}/streamlink/plugins``
Windows           - ``%APPDATA%\streamlink\plugins``
================= ====================================================

.. note::

    If a plugin is added with the same name as a built-in plugin, then
    the added plugin will take precedence. This is useful if you want
    to upgrade plugins independently of the Streamlink version.

.. warning::

    If one of the sideloaded plugins fails to load, eg. due to a
    ``SyntaxError`` being raised by the parser, this exception will
    not get caught by Streamlink and the execution will stop, even if
    the input stream URL does not match the faulty plugin.


Plugin specific usage
---------------------

Authenticating with Crunchyroll
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Crunchyroll requires authenticating with a premium account to access some of
their content. To do so, the plugin provides a couple of options to input your
information, :option:`--crunchyroll-username` and :option:`--crunchyroll-password`.

You can login like this:

.. sourcecode:: console

    $ streamlink --crunchyroll-username=xxxx --crunchyroll-password=xxx https://crunchyroll.com/a-crunchyroll-episode-link

.. note::

    If you omit the password, streamlink will ask for it.

Once logged in, the plugin makes sure to save the session credentials to avoid
asking your username and password again.

Nevertheless, these credentials are valid for a limited amount of time, so it
might be a good idea to save your username and password in your
:ref:`configuration file <cli:Configuration file>` anyway.

.. warning::

    The API this plugin uses isn't supposed to be available on desktop
    computers. The plugin tries to blend in as a valid device using custom
    headers and following the API's usual flow (e.g. reusing credentials), but
    this does not assure that your account will be safe from being spotted for
    unusual behavior.

HTTP proxy with Crunchyroll
^^^^^^^^^^^^^^^^^^^^^^^^^^^
To be able to stream region locked content, you can use Streamlink's proxy
options, which are described in the :ref:`Proxy Support <cli:Proxy Support>` section.

When doing this, it's possible that access to the stream will still be denied;
this can happen because the session and credentials used by the plugin
were obtained while being logged from your own region, and the server still assumes
you're in that region.

For cases like this, the plugin provides the :option:`--crunchyroll-purge-credentials`
option, which removes your saved session and credentials and tries to log
in again using your username and password.

Authenticating with FunimationNow
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Like Crunchyroll, the FunimationNow plugin requires authenticating with a premium account to access some
content: :option:`--funimation-email`, :option:`--funimation-password`. In addition, this plugin requires
the ``incap_ses`` cookie to be sent with each HTTP request (see issue #2088). This unique session cookie
can be found in your browser and sent via the :option:`--http-cookie` option.

.. sourcecode:: console

    $ streamlink --funimation-email='xxx' --funimation-password='xxx' --http-cookie 'incap_ses_xxx=xxxx=' https://funimation.com/shows/show/an-episode-link

.. note::

    There are multiple ways to retrieve the required cookie. For more
    information on browser cookies, please consult the following:

    - `What are cookies? <https://en.wikipedia.org/wiki/HTTP_cookie>`_


Playing built-in streaming protocols directly
---------------------------------------------

There are many types of streaming protocols used by services today and
Streamlink supports most of them. It's possible to tell Streamlink
to access a streaming protocol directly instead of relying on a plugin
to extract the streams from a URL for you.

A streaming protocol can be accessed directly by specifying it in the ``protocol://URL`` format
with an optional list of parameters, like so:

.. code-block:: console

    $ streamlink "protocol://https://streamingserver/path key1=value1 key2=value2"

Depending on the input URL, the explicit protocol scheme may be omitted.
The following example shows HLS streams (``.m3u8``) and DASH streams (``.mpd``):

.. code-block:: console

    $ streamlink "https://streamingserver/playlist.m3u8"
    $ streamlink "https://streamingserver/manifest.mpd"

When passing parameters to the built-in streaming protocols, the values will either be treated as plain strings
or they will be interpreted as Python literals:

.. code-block:: console

    $ streamlink "httpstream://https://streamingserver/path method=POST params={'abc':123} json=['foo','bar','baz']"

.. code-block:: python

    method="POST"
    params={"key": 123}
    json=["foo", "bar", "baz"]

The parameters from the example above are used to make an HTTP ``POST`` request with ``abc=123`` added
to the query string and ``["foo", "bar", "baz"]`` used as the content of the HTTP request's body (the serialized JSON data).

Some parameters allow you to configure the behavior of the streaming protocol implementation directly:

.. code-block:: console

    $ streamlink "hls://https://streamingserver/path start_offset=123 duration=321 force_restart=True"

Supported streaming protocols
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

============================== =================================================
Name                           Prefix
============================== =================================================
Apple HTTP Live Streaming      hls:// [1]_
MPEG-DASH [2]_                 dash://
Progressive HTTP, HTTPS, etc   httpstream:// [1]_
============================== =================================================

.. [1] supports local files using the file:// protocol
.. [2] Dynamic Adaptive Streaming over HTTP


Proxy Support
-------------

You can use the :option:`--http-proxy` option to change the proxy server
that Streamlink will use for HTTP and HTTPS requests. :option:`--http-proxy` sets
the proxy for all HTTP and HTTPS requests, including WebSocket connections.
If separate proxies for each protocol are required, they can be set using
environment variables - see `Requests Proxies Documentation`_

Both HTTP and SOCKS proxies are supported, as well as authentication in each of them.

.. note::
    When using a SOCKS proxy, the ``socks4`` and ``socks5`` schemes mean that DNS lookups are done
    locally, rather than on the proxy server. To have the proxy server perform the DNS lookups, the
    ``socks4a`` and ``socks5h`` schemes should be used instead.

.. code-block:: console

    $ streamlink --http-proxy "http://address:port"
    $ streamlink --http-proxy "https://address:port"
    $ streamlink --http-proxy "socks4a://address:port"
    $ streamlink --http-proxy "socks5h://address:port"

.. _Requests Proxies Documentation: https://2.python-requests.org/en/master/user/advanced/#proxies


Metadata variables
------------------

Streamlink supports a number of metadata variables that can be used in the following CLI arguments:

- :option:`--title`
- :option:`--output`
- :option:`--record`
- :option:`--record-and-pipe`

Metadata variables are surrounded by curly braces and can be escaped by doubling the curly brace characters,
eg. ``{variable}`` and ``{{not-a-variable}}``.

The availability of each variable depends on the used plugin and whether that plugin supports this kind of metadata.
If a variable is unsupported or not available, then its substitution will either be a short placeholder text (:option:`--title`)
or an empty string (:option:`--output`, :option:`--record`, :option:`--record-and-pipe`).

The :option:`--json` argument always lists the standard plugin metadata: ``id``, ``author``, ``category`` and ``title``.

.. rst-class:: table-custom-layout table-custom-layout-platform-locations

============================== =================================================
Variable                       Description
============================== =================================================
``id``                         The unique ID of the stream, eg. an internal numeric ID or randomized string.
``title``                      The stream's title, usually a short descriptive text.
``author``                     The stream's author, eg. a channel or broadcaster name.
``category``                   The stream's category, eg. the name of a game being played, a music genre, etc.
``game``                       Alias for ``category``.
``url``                        The resolved URL of the stream.
``time``                       The current timestamp. Can optionally be formatted via ``{time:format}``.

                               The format parameter string is passed to Python's `datetime.strftime()`_ method,
                               so all the usual time directives are available.

                               The default format is ``%Y-%m-%d_%H-%M-%S``.
============================== =================================================

Examples:

.. code-block:: console

    $ streamlink --title "{author} - {category} - {title}" <URL> [STREAM]
    $ streamlink --output "~/recordings/{author}/{category}/{id}-{time:%Y%m%d%H%M%S}.ts" <URL> [STREAM]

.. _datetime.strftime(): https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes


Command-line usage
------------------

.. code-block:: console

    $ streamlink [OPTIONS] <URL> [STREAM]


.. argparse::
    :module: streamlink_cli.main
    :attr: parser_helper
