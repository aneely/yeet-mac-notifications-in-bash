_Keep Your Hands on Your Keyboard and Keep Going_

## TL;DR:

- You can use AppleScript to manipulate the GUI layer of macOS and the `osascript` command can run it from the terminal.
- I've adapted two examples of AppleScript to dismiss banner notifications and clear out Notification Center.
- They work when run from Script Editor and could be turned into keyboard shortcuts with Automator.
- I want to use the terminal instead, so I've wrapped them in shell scripts that can even be run as commands. 
- If you'd like to dismiss nagging banners or empty the notifiations list, all while keeping your hands on the keyboard so you can stay in the terminal, this worked for me.
- Caveats and "But, why?" further down this page, as are some rationalizations, excuses, and links.

## The Scripts

### Dismiss system alerts and banner notifications 

Note: this works by clicking the first button on the banner or alert. Keep that in mind depending on your alert buttons. I found this on [Stack Exchange](https://apple.stackexchange.com/a/155736).

```
#! /usr/bin/env bash

osascript << 'END' &>/dev/null
my closeNotif()
on closeNotif()

tell application "System Events"
	tell process "Notification Center"
		set theWindows to every window
		repeat with i from 1 to number of items in theWindows
			set this_item to item i of theWindows
			try
				click button 1 of this_item
			on error
				
				my closeNotif()
			end try
		end repeat
	end tell
end tell

end closeNotif
END
```

### Clear out Notification Center on Mojave

Note: this will also clear any active banner notifications that also appear in the notifications tab. This is adapted from another [SE answer](https://apple.stackexchange.com/a/374455).

```
#! /usr/bin/env bash

osascript << 'END' &>/dev/null
tell application "System Events"
	
	# Show "Notifications" Tab
	tell application process "SystemUIServer" to click menu bar item "Notification Center" of menu bar 1
	
	tell process "Notification Center" to click radio button 2 of radio group 1 of window 1
	
	# loop through the app close buttons
	tell process "Notification Center" to try
		repeat -- forever (at least until there are no more)
			delay 0.25
			click UI element 2 of UI element 1 of row 2 of table 1 of scroll area 1 of window "Notification Center" -- the topmost close button
		end repeat
	on error errmess -- no more
		log errmess -- uncomment for troubleshooting
	end try
	
	# close the window
	tell application process "SystemUIServer" to click menu bar item "Notification Center" of menu bar 1
	
	# Return to "Today" Tab
	tell process "Notification Center" to click radio button 1 of radio group 1 of window 1
	
end tell
END
```

## Caveats

- The words in this page make sense individually but not necessarily together or in this order.
- Pasting strings of text from random websites into a shell and running them is a terrible idea that I definitely don't do all the time. (I usually skim them first.)
- Seriously though, sanitize text from your clipboard before it touches your shell if you copied it from a browser.
- Don't do this to a production Mac in a multiuser environment. If you do this and then blame me, I'm going to turn off all the lights and pretend I'm not here when they knock on my door.
- You will probably need to set some system Accessibility permissions under Security and Privacy on your system the first time you run one of these and then run it again for either to work.
- If you can't administer these permissions, say on a work computer, this approach might not work for you. You can thank Apple's comedic impression of Windows Vista for that.
- Works On My Machine, but I'm not on Big Sur. See the very end if you are and this doesn't work for you.
- All code is covered under the GPHL, or Giant Pile of Hacks License, which guarantees nothing more than hacks, which may or may not be piled up to any specific size, gigantic or otherwise.

## Valid Excuses for Common Concerns

### But, ...why?

I'm trying to spend more time staying in the terminal and focusing on work, and dismissing notifications with the cursor was getting in my way.

### You know you can just reach up and...

Let me stop you there. Swiping away the banner takes my hands off the keyboard, as does clicking on the banner's dismissal button. Clicking on the banner itself is only useful when I want to break out of what I'm doing by letting another app or browser tab steal focus. (Looking at you Google Calendar.) When I'm working, I just want to clear my screen and keep going.

### Why the terminal, nerd?

If I'm at work and my hands are on the home row, I'm either in iTerm or can switch to it immediately, so a terminal command works perfectly for me. A script can be made executable, e.g. `chmod +x "$scriptname"`, and dropped in `/usr/local/bin` to make it availabe in my `$PATH` as a command. (Maybe just leave the conventional `.sh` off the end of the filename.)

### What's that cursed content on line 3?

Much like the way Mark Zuckerberg's face keeps you from seeing the front of his skull, this syntax doesn't look elegant but it does the job. If you don't live in `bash`, `osascript << 'END' &>/dev/null` is doing a few things at once they need to happen on the same line.

`'END'` pairs with the bare `END` of the heredoc at the end of the script that's letting me write the AppleScript in multiple lines with indentation like a person with dignity who eats with utensils.

`<<`, or special redirect, passes it to `osascript`. Putting `'END'` in quotes prevents `bash` from trying to interpret anything in that block, like variables, commands, or unescaped special characters, which we don't want.

`&>/dev/null` is there because AppleScript is implicitly logging to standard out and standard error. I couldn't find a flag to shut this off in the `osascript` man page. I don't want to see it so off to the void it goes.

### Why is this page?

You can throw a rock and hit dozens of examples of how to display notifications with a script, but there was very little about how to script getting them off your screen. I'm leaving my solution here, along with some links to where I found the code I adapted, because I was kind of surprised by how little this seems to be documented online.

### Yeah but why is this a page and not a blog post?

Why is your face a... never mind. The moment's passed.

### Why not just change the notification preferences?

I do need to see and read _some_ work-related alerts, so this is pretty closely scoped to dismissing them after that happens. A better approach for actual peace and quiet would probably be using Do Not Disturb more often, scheduling stretches of time where notifications don't show up, or even disabling system notifications entirely. But those are solutions for a slightly different problem space.

### But Automator!

Automator is a totally valid approach and a system-wide keyboard shortcut that runs the AppleScript is the most convenient solution there. I'm more interested in something that's easy to git-clone into a terminal setup on different machines.

### But AppleScript nitpick!

If you want to share some AppleScript, please feel free! While I probably won't ever do a deep-dive into being an AppleScript expert, packing bits of it up like this to trigger things in GUI land is turning out to be useful. For now, I'm just adapting other peoples' code enough to turn this into a problem I can solve from `bash` or `zsh`. Throwing AppleScript code at `osascript` is a means to an end.

### This doesn't work for me

The license covers hacks. Nobody said anything about working hacks. But if you're on Big Sur, check the end of the page.

### You can do this so much easier with JavaScript

Great! [Let's talk](https://twitter.com/ravinglogic).

### But, like, _why?_

The default behavior of many banner notifications is to stick around obstructing very active parts of my screen until I've dealt with them. In practice, even when a notification has useful and timely information, like a meeting starts in 5 minutes, I don't want it sitting there _long_ after I've noticed and read it.

And some apps are worse about this than others. Anything involving scheduling or the system is especially insistant about hanging around. (Looking at you Calendar.app and Google Calendar.)

It's hard problem to design for broadly and notifications have to distract to do their job. But I need concentration to do my job, and continuing what I'm doing until I'm ready to switch needs to win.

### What do have to say for yourself?

Not much and not often, but I'm [@ravinglogic](https://twitter.com/ravinglogic) on Twitter.

## ~~Random~~ Bonus Content

### JavaScript code for an Automator action for BigSur

I also don't have an M1 Mac, so being on Big Sur isn't mandatory or even desireable for me even on my personal equipment since everything works and Mojave is still getting security updates.

But I'm tucking this here because I probably will have one at some point and [some people](https://apple.stackexchange.com/a/413495) on Stack Exchange are reporting the AppleScript to clear notifications doesn't work, but supposedly this [gist of JavaScript code](https://gist.github.com/lancethomps/a5ac103f334b171f70ce2ff983220b4f) does as a step in an Automator action. Plus it's still fresh as of late Feb 2021.

`osascript` will accept JavaScript code with the right flag: `osascript -l JavaScript -e 'console.log("Hello world");'` so that would be my preferred approach to try. But some quick and dirty instructions for running JS code in Automator are [here](https://support.apple.com/guide/automator/add-javascript-scripts-to-a-workflow-auta229c77c2/mac) if that's your jam.

### System prefs for shortening how long notifications are presented on macOS

Maybe you can't grant the needed permissions to macOS for the scripts to work. If you're in the sudo group, this might work instead to make the problem less bad. According to (yet another) [SE answer](https://apple.stackexchange.com/a/64029), running this in the terminal will set the system preference for how long to show notifications: `defaults write com.apple.notificationcenterui bannerTime <SECONDS>`
