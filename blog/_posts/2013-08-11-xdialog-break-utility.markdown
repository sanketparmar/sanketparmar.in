---
layout: post
title: Break reminder utility
last_changed: 2013-08-17
published: true
tags: break utility Xdailog cron linux
---

It's a bad habit to use computer for long hours. It is not good for your eyes.
Also it decreases your productivity. I have this bat habit too and I need some short
of tool to remind me my breaks. The basic features I need from this tool are

1. Easy plus flexible scheduler
2. Block my screen for pre-configured time
3. Status bar or countdown time

First I thought to use some ready made tools, but then I decided to write my own
to learn something new.

I made one simple utility using `Xdialog` and `cron`.

### What is Xdialog ?
[Xdialog](http://xdialog.free.fr/) is designed to be a drop in replacement for the "dialog" or "cdialog"
programs. It converts any terminal based program into a program with an
X-windows interface. The dialogs are easier to see and use while adding even
more functionalities (e.g. with the treeview, the file selector, the edit box,
the range box, the help button/box). Because Xdialog uses GTK+, it will also
match your desktop theme.

To schedule a task, I used cron job.

{% highlight bash %}
#!/bin/bash
#
# Total time to wait in secs
total_time=300
#
# Time to update the progress bar, in secs
sleep_time=10
#
# Put your favorite quote script
quote=`fortune`

#
# To start gui application from cron, Set DISPLAY
if [[ -z $DISPLAY ]]; then
	export DISPLAY=:0
fi

start_time=$SECONDS
diff=0
while [[ $diff -lt $total_time ]]; do
	sleep $sleep_time
	diff=$((SECONDS-start_time))
	echo $(((diff*100)/total_time))
done | Xdialog --title "Take a break... Have some coffee... You deserve it :)" \
		--left --guage "$quote" --fill -1 -1
{% endhighlight%}

You can grab the script from [here](https://github.com/sanketparmar/toys.git).
Set the value of  `total_time` as per your desire.

Now to schedule the a task, execute the following commands.

	$ crontab -e

This will open an text editor. Copy and paste following line and save the file.

	00 09-18 * * 1-5	/path/to/your/script

This cronjob will execute the script hourly from 9 to 6 on weekdays. Change this
cron setting  as per your requirement.

Final outcome
[![Break Reminder](/images/break_reminder.png =640x)](/images/break_reminder.png)

TODO: I would like to add snooze button in this script.
