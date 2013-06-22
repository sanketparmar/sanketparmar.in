---
layout: post
title: Small Linux Commands - yes
last_changed: 2013-06-18
tags: yes linux command automation
---

## Linux command: `yes`
   
yes - Repeatedly output a line with all specified STRING(s), or 'y'
   
#### Where to use ?
 
To sends a default response to any command. Many Linux commands ask for confirmation for performing an action. For example `rm` ask for confirmation before deleting. you can achieve same behaviour using -f switch of `rm` command. However there are many commands which doesn't have this capability.  
<br/>
   
Some examples:    
>`yes | rm  *.txt`    
 Delete all txt files without any prompt (same as `rm -f *.txt`).

>`yes "abcdefg" > sample_data.txt`
To generate sample data files for testing.

>`yes "pass" | my_test_application`
To provide pass for each prompt.  
      			
You can check your machines performance on high load using multiple `yes` commands in different terminal(or run them in background by appending `&`).

