---
layout: post
title: Vim Tips - Save file with sudo permisson
last_changed: 2013-08-11
published: true
tags: vim linux sudo
---

![Vim Permission Error](../../../images/vim-permission-error.png)

What do you do if you open a read-only, protected file using `vim` and want to 
save the the modifications. 

Solution 1. Discard the changes, reopen the vim with sudo and modify the file again.

Solution 2. Save the file to some temporary location. Replace original file with temp file.

Solution 3. Use following command.

	:w !sudo tee % 

I know you like the third solution. But what the heck!! How does it work?

Vim stores each and every changes into buffer and if we type `:w`, It tells
Vim to write the current buffer into the current file. `:w copy_of_test.txt` writes
current buffer into copy_of_test.txt file. Like wise `:w !{cmd}` will execute the 
cmd command and pass the current buffer as standerd input. Vim also rememberes the 
"current file name".  This is also known as the name of the current buffer. 
It can be used with "%" on the command line. [Source: Vim Help](http://vim.org)


Here our commad is `sudo tee current-file`

Content of current buffer is proived to `tee curent-file`. It reads from standard input 
and write to standard output and file with superuser permission.  

![Vim Load File](../../../images/vim-load.png)

Vim will ask you to load the modified file. Press "L" to load.

You can also compress the extra output by redirecting the stdout to /dev/null.

	:w !sudo tee % > /dev/null

It's a good idea to create a shortcut for this in your vimrc file. Mine contains
following line

	" Trict to write as sudo                                                 
	 map sw <ESC>:w !sudo tee % > /dev/null<CR>

Now Just press s + w  to store the file with sudo permission. 

Happy Edititng :)
