fmShellScript
=============

Haven't you ever wanted to do some Perl, BASH, Ruby, or Awk on your
FileMaker data? Don't you get tired of constantly having to do
multi-step file exports (export the file, run your script on the file)
to make this happen? Don't you wish you could write generic reports that
would work on *any* FileMaker export in a similar format? Sometimes,
maybe, wouldn't it be nice to run some AppleScripts on FileMaker data,
without all of that messy quoting and escaping? This module is the
solution to those problems.

The problem with running shell scripts in FileMaker is that there's only
one way to do it: the AppleScript “do shell script” step. If you've ever
tried this from within FileMaker, you know it's a huge pain to get the
quotes and everything escaped properly. Wouldn't it be nice if you
didn't have to worry about that, and could focus on the code you're
writing instead?

What `fmShellScript` does is take a shell script specified *inside*
the database, save it to a place you specify on your computer (in the
proper ASCII format), and then run the script using an AppleScript
wrapper, optionally using a `«parameter»` that you can send using an
API, or specify right in the database. This allows you to send FileMaker
data to a shell script (typically modifying an exported file, but any
parameter is allowed) or an AppleScript, without having to leave
FileMaker.


Structure
=========

Each record specifies:

-   A script title and category. Title *must* be unique, and category
    should be something useful. Reuse category where appropriate.
-   The contents of a shell script. This can be any script executable
    from the shell - BASH, Perl, Ruby, Awk. You could even write a C or
    Java program if you wanted to get fancy, but you might have to put
    it in a script wrapper.
    -   Click `[edit in external editor]` to, you know… The “external
        editor” defaults to “TextWrangler”, but you can change the
        default for any script.
    -   Click `[export to path_to_script]` to save the script to your
        computer.

-   A `«path_to_script_eval»` field that evaluates to the path where the
    script file will be saved on your computer.
    -   You can reference this value in your AppleScript calculation
        with `«path_to_script»` (OS X URL) and `«path_to_script_posix»`
        (POSIX path - no computer name).

-   An AppleScript calculation that `fmShellScript` will use as a
    wrapper to run the shell script
    -   Use `«double-bracket syntax»` for performing FileMaker functions
        in this calculation. For example, `«1 + 1»` would run in Script
        Editor as `2`
    -   This is where you pass your variables to the shell script using
        the “do shell script” command (see “usage” below)
    -   Hover over this field to see the actual AppleScript to which it
        will evaluate.
    -   Click `[copy applescript]` to copy the AppleScript evaluation to
        the clipboard. Useful for copy/pasting into Script Editor/XCode
        for debugging.
    -   Click `[perform applescript]` to run the evaluated AppleScript.
        If you have specified a `«parameter»` above, that parameter can
        be referenced inside your AppleScript.
        -   When you call the script from an external API script, the
            parameter specified locally can be replaced with data in the
            API call. If `«parameter»` isn't specified in an API call,
            whatever is in the record will be used as the script
            parameter.


Usage
=====

1.  `fmShellScript` is either called remotely using the API script
    `“pub - Run Shell Script ( script_name { ; parameter } )”`, or run
    locally using the `[perform applescript]` button or “Open Script”
    function (see below).
2.  If calling externally, you send the API script the name of the
    script you wish to run (such as “TSV to HTML”), and an optional
    parameter. If calling locally, the `parameter` specified at the
    top-right (internally known as the `“test_parameter”`) is used.
    -   Note that a passed parameter might not be necessary depending on
        the script.

3.  `fmShellScript` will copy the script to the path specified in
    `«path_to_script»`, then run the applescript.
4.  You set up the AppleScript to run your script file on your
    parameter. The default template for a new record sets this up for
    you.
    -   However, you could make this AppleScript do *anything* if you
        wanted to. Chimes, bells, system alerts, your taxes, etc.

5.  It's up to you to make sure that the script is doing what you want
    (ie placing a file on the desktop, or sending it somewhere, or
    what-have-you).


"Open Script" Function
======================

`fmShellScript` has a “Open Script” mode, where you can get a list of
scripts, by category, select a script, and then call it using a
user-specified parameter or on a file. To use this function, just call
the script remotely using the
`“pub - open the “select a script” window”` API script.

Note that this view stores user “preferences” like last category
selected, last script selected, and last parameter used. It stores users
in a table based on a combination of account name, and IP Address. That
way, if you decide not to define accounts, everybody still gets their
preferences stored by IP address. If you *do* decide to define accounts,
a user will have different preferences depending on the machine or
portable device they are using.


Notes
=====

Clearly, this database only works on a Mac. Sorry about that Windows
people.

To help you in implementing shell script exports in your solution, there
is a SAMPLE SCRIPT or two in the script menu.

You can click on the `«path_to_script»` or `«path_to_script_posix»`
field labels to open a finder window showing the folder in which they
are contained.

You can click on the `«parameter»` field label to select a file to be
used as your parameter. Note that a parameter doesn't *need* to be a
file - it's up to you what parameters your script takes.

There are some helpful custom functions in setting up pathnames:

-   `GetPath(path_to_file)` - returns the `/path/to/` portion of
    `/path/to/file`
    -   for example, if your script was “cat $1”, and you wanted to save
        its output to a file, your AppleScript could be the following:

    do shell script "/bin/sh «path_to_script_posix» «path_to_passed_file» «GetPath(path_to_file)»export.html"

-   `GetFileName(path_to_file)` - returns the `file` portion of
    `/path/to/file`
-   `POSIXPath(path)` - strips computer name from path
    -   for example, `«Get ( DesktopPath )»` might return
        `/Macintosh HD/Users/bozo/Desktop/`
    -   `«POSIXPath ( Get ( DesktopPath ) )»` would then return
        `”/Users/bozo/Desktop/”`
