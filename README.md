# Tinderly
A tinder bot can automatically swipe right for you and even send automated messages if you really want to get fancy. However, I should warn you with the typical disclaimer. Any attempts to spam or harass anyone through the use of a bot are not condoned  by myself or Tinder. It could lead to Tinder shutting down your account or worse. What you can do is minimize the amount of work needed to play the dating field, increase your chances of getting matches by auto-swiping, and use data to optimize your swipes, messages, and dates. This post is only for educational purposes. Now let’s get started.

You’ll only need two (plus a third optional) things to build a Tinder bot:

    Facebook+Tinder authentication
    Burp (https://portswigger.net/burp/download.html) (optional)
    Python with Pynder installed

(NOTE: THIS NEXT SECTION IS NOT REQUIRED, GIVEN THAT I HAVE INCLUDED A FUNCTION FOR AUTOMATING GETTING AN AUTHENTICATION TOKEN (Tinder Token Retriever) IN THE LATEST VERSION OF THE BOT POSTED BELOW.)

First, to set up authentication, use your desktop web browser (I would recommend Chrome) to go to the Facebook profile that will be used with Tinder. Right click the Facebook profile web page and select “View Page Source”.  Then, command+f (or ctrl+f for Windows users) to find “profile_id” in the source code. Save the value listed for the profile_id. This will be used as the FACEBOOK_ID for authentication.

Now,  if you have already signed into Tinder through this Facebook profile, then you’ll need to go to the Facebook Settings and select the Apps settings tab. Then, remove Tinder from the Apps logged in with Facebook. Once you’ve removed Tinder, on that same browser tab, right click and select “Inspect”. Then in the browser URL bar, paste this URL: https://www.facebook.com/dialog/oauth?client_id=464891386855067&redirect_uri=fbconnect://success&scope=basic_info%2Cemail%2Cpublic_profile%2Cuser_about_me%2Cuser_activities%2Cuser_birthday%2Cuser_education_history%2Cuser_friends%2Cuser_interests%2Cuser_likes%2Cuser_location%2Cuser_photos%2Cuser_relationship_details&response_type=token&__mref=message.  Click “Continue” when prompted. Now, go back to the “Inspect” window and we’re going to look for the authentication token. Go to the “Network” tab. Select “XHR” for the filter. Then look for one named something like “read?dpr=2”.  If you click on it, you should see some stuff in the “Preview” window. Click the arrow next to “jsmods” in the “Preview” window. Keep clicking through the nested arrows (“require”, “0”) until you see a final arrow (possibly next to a number like “3”). Click this final nested arrow and there should be a url like “fbconnect://success#access_token=” with a token behind it. Save that token. It will be used as the FACEBOOK_AUTH_TOKEN for your bot.

(OPTIONAL) Second, you’ll can install Burp on your desktop/laptop. Burp is a tool used to intercept requests and responses through a proxy, acting like a middle man, allowing you to see how a website or app is operating under the hood. Once you have Burp installed on your desktop/laptop, you’ll need to configure it so that traffic from your phone is piped through a proxy set up in Burp. Burp has plenty of good documentation on how to set this up:

    Configuring an iOS Device to Work with Burp (https://support.portswigger.net/customer/portal/articles/1841108-configuring-an-ios-device-to-work-with-burp)
    Configuring an Android Device to Work with Burp (https://support.portswigger.net/customer/portal/articles/1841101-configuring-an-android-device-to-work-with-burp)

Once you’ve configured your mobile device to run through Burp’s proxy, there are just a few other options you’ll want to set up. In the Proxy tab under Options, make sure all the boxes are checked for Intercept Client Requests and Intercept Server Responses. This will ensure everything needed to build the Tinder bot is intercepted.

Now you’re ready to start seeing how Tinder requests and responses are formatted. With “Intercept” turned on in Burp on your desktop, open Tinder on your mobile device. You should see information populate in Burp. If you want to explore the different requests and responses, you can hit the “Forward” button to allow the actions to go through as you use Tinder. You can use Burp to reverse engineer virtually any API, including Tinder’s. So, should Tinder ever make any changes to their API, or should the way they authenticate to Facebook change, you can always come back to Burp as a tool for helping determine what changed and how to adjust.

Using Burp is not required for this Tinder bot to work, but it is a tool that will prove forever useful if you want to explore adjusting your bot or building other bots. For the purposes of the next steps, you can keep Burp turned off and your web browser or mobile device running without any proxy. This will allows Tinder’s API requests/responses and your browser to work as normal.

Essentially, our Tinder bot will operate by authorizing itself, and then mimicking the Tinder API requests/responses through Python code. Luckily, there is a Python library called Pynder that makes setting this up easy, which brings us to our third step:

    Pynder (https://pypi.python.org/pypi/pynder/0.0.11)

Assuming you already have Python and pip installed, you can install Pynder by simply opening your terminal and typing sudo pip install pynder. Now let’s get into the actual Python code.

The flow structure of our Tinder bot will go like this:

    Start a Tinder session by authorizing
    Check if we have reached our swipe limit
    If we are out of swipes, move along to send messages to the matches (steps 7+)
    Else, auto-swipe right (except randomly swipe left 1 time out of every 100 swipes to seem more human).
    Sleep between 1 and 5 seconds at random amounts in between each swipe (to seem more human).
    Continue swiping until out of swipes, then move along to send messages to the matches.
     Grab the list of matches.
    For each match, grab the list of messages already in our inbox.
    If match isn’t already in our messages, send them the first message to start a conversation.
    If the number of messages already sent to the match is greater than or equal to the length of our list of messages to send, log the conversation is finished with that match.
    Else, if the latest message is from myself, log that there are no new messages from that match and move along to the next match.
    Else, if there is new message from someone and it contains the words “bot”, “fake”, or “real person”, then send the appropriate responding message (we have different message lists for each word to make the conversations somewhat dynamic).
    Else send the next generic message. Then move along to the next match.
    Do this for each match in the list of matches.
    Log resting for a few minutes and sleep a random amount between 300 and 600 seconds (to seem more human).
    Repeat.
