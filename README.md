# rfc6238tok
a Python3 script to generate RFC 6238 authentication tokens

This was inspired by a Java program called "JAuth".  While I've been using that for years, it lacked certain features...such as labelling what the auth code is for (Google, Github, etc.).  (Yes, I know version 2 keeps track of several IDs and has a "PIN", but the way that works was also rather unsatisfying.)

This explicitly does NOT deal with how you store and retrieve your "seeds" for generating the tokens; that's up to YOU!  Personally, I have a directory with a collection of files, each with a suffix .seed.gpg.  As you may have guessed, I run [GNU Privacy Guard](https://gnupg.org) to decrypt their contents, and that feeds a line of text into this script.

When invoked with an argument, that argument is used for a label in the window, as well as being incorporated into the window's title.  You feed your base-32-encoded secret/seed as a single line into the stdin, usually with a pipe. **_PLEASE NOTE CAREFULLY THAT THIS DOES NOT USE TERMIOS() OR ANYTHING ELSE TO TURN OFF ECHO, SO IF YOU DO NOT PIPE IN DATA, YOUR TERMINAL WILL VERY LIKELY ECHO YOUR SECRET_**  I chose to pass data in this way because environment variables could be snooped by ps(1), and files have problems all of their own (told you, it's not the purpose of this piece to manage your secrets; that's explicitly your problem).  Even though ordinarily you'd feed the data from a pipe into this program, it still reminds you what it wants by outputting a prompt.  (A number of times I would run the program and think it had problems because it did nothing.)  To be compatible with the .rc file for JAuth, if what is fed in begins with "secret=", that is stripped off.  (Almost) **NO error checking** is performed on your input, Python will just quit and if you invoke this in a terminal, you see an error backtrace.  One exception is, it will make sure the string length is a multiple of 8, and pad with "=" if it's not.

It may not have perfect timing, but most services that use these codes have timing slop, not only in terms of seconds synchronization, but also plus/minus whole windows over which codes will be valid. So use with caution.

After passing your secret in, a Tk window will be opened.  Whatever argument you give this program will be incorporated into the window title as well as showing up as a label in the window.  The auth code will appear in Courier New as green on black, with a copy button next to it.  Clicking on "copy" attempts to copy the current code to the clipboard as well as trying to own the X11 selection.  Below that will be a progress bar updated approximately once every second.  It starts out green at :00 and :30.  From 10 seconds to 5 seconds left in the likely validity of the code, the progress bar becomes yellow.  While there are 5 seconds or less of code validity, the bar turns red.

As a visual indicator, when "copy" is clicked, the background of label above the code turns green.  If some other program takes away ownership of the selection, that background turns yellow.

A few words about the clipboard and the selection:  When you click on "copy", the current 6 digit token replaces the contents of the clipboard and we take ownership of the selection.  If the current token expires, provided nothing else has manipulated the clipboard, it will still have the expired token in it.  If nothing else takes ownership of the selection, and you paste the selection (for example, in most system configurations, by pressing the middle mouse button), whatever the current token is gets pasted; there is no need to click "copy" again (although nothing is lost if you do, so click away if you want).  So it matters what paste method you use, clipboard or selection.  Some systems (MS Windows?) might not make a distinction between clipboard and selection, so you'll have to use the copy button.

Here is an example of usage:

    gpg2 --decrypt < ~/.config/mfaseeds/google.seed.gpg | rfc6238tok Google

In my environment, gpg2 will automatically invoke gpg-agent and put up a graphical prompt for the file's encryption passphrase.
