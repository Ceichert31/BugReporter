# BugReporter
The BugReporter allows players to send messages using a discord webhook.  
This allows easy and low maintanance feedback from directly inside your game.  
Webhooks can be easily abused for anonymous automated spam or simply overwhelmed. That makes this a bad solution for games with many players.  
Tested for Godot 4.0.3 4.1.3 4.2.2 4.3 4.4.1 and 4.5 (All versions I still had installed).  

# Config
This is an example config with all needed values. Only the url is needed. game_name will default to "unnamed_game" if not specified.

```
[webhook]

url="https://discord.com/api/webhooks/<webhook.id>/<webhook.token>"
game_name="BugReporter"
tts=false
color=15258703
anonymous_players=false
send_log=false
send_analytics=true
```

The Reporter Node also gives the option to change the path to the config file, disable auto closing and disable auto clearing for inputs.

You don't need to include the config in your export settings. The addon comes with an ExportPlugin that includes your webhook config automatically and gives you error messages on export.

<span style="color:red">**WARNING:** The webhook url can be used to read all messages send with the webhook.  
Do NOT use the url directly if you allow sending any non anonymous information.  
Using the url for anonymous messages is still highly discouraged.  
See Proxy section for save usage.
</span>

# Example usage

![](screenshots/reporter_ui.png) ![](screenshots/message_with_screenshot.png) ![](screenshots/send_screenshot.png)

The playerid depends on the Game name and device id. This allows you to search reports by player but they are still mostly anonymous. You can disable this entirely with `anonymous_players=true`  
If not provided Contact Info will not be displayed.  
You can add or remove as many Categories as you like by editing the OptionButton of the Reporter.  
F2 takes a screenshot (configurable with the input action `screenshot`). The screenshot can be attached to the message.  [See Screenshots for details](#screenshots) 

# Analytics
Most of the time a screenshot is not enough to figure out what happened in a bugreport.  
To help with that you can allow your players to send analytics with their message.  
BugReporter supports 2 kinds by default.  
You can include any kind of data, use this power responsibly. Take care to only send relevant information (redact sensitive data if needed) and only with the players's consent. Anything else is negligent at best and criminal at worst.
## Report
The Analytics Report will look something like this.  
```
Report-1692997835-Linux-16-3

root/Example
Example:<Node2D#27682407576>

root/Example/Player
pos: (539, 55), texture: res://icon.png
```
It starts with the Header `Report-Timestamp-OS name`  
Then it lists all nodes in the `analize` group by printing their full node path and the result of their `analize()` (Like root/Example/Player) function if they have one or the result of `_to_string()` (Like root/Example).  


This can be enabled with `send_analytics` in the config file.
## log files
For an easier to setup variant you can also just send the godot.log file. Again you should beware of data privacy. Do not use this if your log contains sensitive information.

This can be enabled with `send_log` in the config file.
Make sure to enable `application/run/flush_stdout_on_print` in the Project settings. Otherwise the file will not always be written instantly. Then the log might miss lines at the end or be completely empty.
Also set `debug/file_logging/enable_file_logging` to make sure logs are written on non desktop platforms if needed.

## log files vs Report
What is best to use depends on the game.  
The log is a timeline of everything that happened. The Reporter is a snapshot of the states of every relevant node.  
choose one or both.

# Screenshots
This addon is fully compatible with [ScreenshotManager](https://github.com/ASecondGuy/ScreenshotManager). 
The default and subnautica like Reporters can take and send screenshots, but they only handle the most recent screenshot and don't save them anywhere.
With the ScreenshotManager plugin installed they automatically disable their internal screenshot handler and use its improved functionallity.  
Including central screenshot management and selecting any previously taken screenshot.

# Customisation
## Message
The looks of the final Message can be entirely customized by changing the `send_report()` function in the `reporter.gd` script and using the `WebhookBuilder` provided with `_webhook`. 
## UI
The Reporter UI can be easily customized as long as the correct nodes are still provided in the main `Reporter` node.
## More specific questions
More information, questions etc can be freely added by first adding the input nodes (like buttons, LineEdit, etc.) and then adding the information to the message by adding a field, embed or whatever.

Examples:
```gdscript
  # How the reporter attaches the log file and Analytics
  if attach_log_file:
    _webhook.add_file("user://logs/godot.log")
  if attach_analytics_file:
    _webhook.add_file(AnalyticsReport.new(get_tree()))
  
  # How you could attach the savefile
  if attach_save_file:
    _webhook.add_file("user://savefiles/save%s.sav" % Autoload.loaded_savefile)
```
```gdscript
	# embed basics
	_webhook.start_embed()
	_webhook.set_embed_title("%s by %s" % [_options.text, player_id])
	_webhook.set_embed_color(_cfg.get_value("webhook", "color", 15258703))
	
	# How the reporter attaches contact info
	var contact_info := _mail_line_edit.text
	if !contact_info.is_empty():
		_webhook.add_embed_field("Contact Info:", contact_info)
  
	# How you could attach some other line edit input like a favourite character.
	var fav_character := _fav_character_line_edit.text
	if !fav_character.is_empty():
		_webhook.add_embed_field("Favourite Character:", fav_character)
	else:
		_webhook.add_embed_field("Favourite Character:", "none selected")

```

## Example subnautica like
As example for an alternatve you can look at `subnautica_like.tscn`.  
It more closely resembles the feedback form in subnautica that inspired this addon.  
![](screenshots/subnautica_like_ui.png) ![](screenshots/subnautica_like_message.png)  

## Example survey
If you want feedback as more of a survey style format you can use `survey.tscn`
![](screenshots/survey_start.png) ![](screenshots/survey_question.png) [](screenshots/survey_message.png)  
Supports translations, optional questions, required questions, custom greeting & goodbye, and easily customized questions and answers.

For reference this is the default survey with explenations. Some questions and answers are intentionally absurd to encourage you to make your own.
```
# Customizing these questions is highly encouraged
# Lines starting with #  and empty lines are ignored
# Every question or answer can be a translation key. They will be auto translated.
# This whole field can be a translation key to customize the entire survey for different languages.

# Lines starting with ? are Questions
?How did you learn about this game?
# Other lines will be the possible answers
Friends
Youtube
Discord
Itch
Steam
Fortnite

?Before I started to play I ...
	# you can indent however you like for readability
	didn't know what the game was about
    wasn't interested in the game
	was interested in the game
	was very interested in the game

			?Now I am ...
	interested in the game
    very interested in the game
   no longer interested in the game

?I will replay the game...
	soon
	never
	after a long time
	regularly

?Did you like the soundtrack?
	
	No
	A little
	Yes
	I'd like a cassette of it please

?Did you like the Artstyle?
	No
	A little
	Yes
	I'd like some merch of it please

?Will you recomend this to your friends?
	Yes
	Totally
	Absolutely

?Would you pay for this game?
	Nothing
	1$
	5$
	10$
	20$
	30$
	60$

# Some questions are too important to leave unanswered
# Mark them with ! instead of ? and they become unskipable 
# You could also make all questions required and offer "Prefer not to answer" options
!Do you have male pattern baldness?
	Yes
	No
	I hope so
	I hope not
	Prefer not to answer

?Unfortunately, the clock is ticking the hours are going by. The past increases, the future recedes. Possibilities decreasing, regrets mounting. Do you understand?
	I understand
	remain ignorant

```


# Proxy
To prevent data leaks and potential data privacy lawsuits you need to setup a proxy.
A Proxy will relay any messages to discord while ignoring unwanted actions (reading, deleting etc.).  

The recomended way is to use a cloudflare worker with this code. Then put the proxy url in the webhook.cfg instead of the webhook url.
```js
export default {
  async fetch(request) {
    const destinationURL = "webhook url";
    if (request.method === "POST") {
      const mod = await fetch(destinationURL, request);
      return new Response(null, {status: mod.status});
    }
    return new Response("Only POST is allowed", { status: 403 });
  },
};
```

Make sure the webhook url is only in this code to make sure nobody malicius can get a hold of it.  
If you want to implement a content filter or spam prevention this would be a good place to do it.