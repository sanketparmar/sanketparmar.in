---
layout: post
title: Get Unread Mails From Google Using Gmail - ATOM Feed
last_changed: 2011-11-28
published: true
tags: gmail atom conky Perl
---

Gmail has a very cool feature not available in other webmail applications:
feeds for unread messages. Gmail supports Atom feeds. Atom is used to easy for
you to receive, in one place, regular updates from news websites, blogs. In this
post we will see how to access gmail atom feed and use its contain in conky.

You can simply access the gmail atom feed using [https://mail.google.com/mail/feed/atom](https://mail.google.com/mail/feed/atom)
url. Enter your gmail user name and
password. It shows most recent unread message of your inbox. Now if you want
unread message from any of your gmail labels, then just use
[https://mail.google.com/mail/feed/atom/lablename](https://mail.google.com/mail/feed/atom/lablename).
You can provide your username and password in url also. For example
https://username:password@mail.google.com/mail/feed/atom/.

The feed is limited to only the last 20 new messages. If you want to more then
you have to use IMAP or POP protocols to connect and fetch mails from gmail server.

Here is simple Perl code which reads the gmail atom feed and parse the output.
I am using this script to display my mail information in conky configuration file.


{% highlight perl %}
#!/usr/bin/perl
use strict;
use XML::Simple;
use LWP::Simple;

# use name
my $u = $ARGV[0];
# password
my $p = $ARGV[1];
my $xmldata = get("https://$u:$p\@mail.google.com/mail/feed/atom");

# create object
my $xml = new XML::Simple;

# read XML file
my $data = $xml->XMLin($xmldata);

my @arr = reverse sort keys %{$data->{'entry'}};

if (@arr != 0 ) {
print "-----> [$u] New mails: ". @arr ."\n";

# Max no of mails you need to display
my $n = 3;

$n = @arr if @arr < 3;
for(my $i = 0; $i < $n; $i++) {
    # Get the Title
    my $title = substr $data->{'entry'}->{$arr[$i]}->{'title'}, 0, 45;
    # Get the from field
    my $frm = substr $data->{'entry'}->{$arr[$i]}->{'author'}->{'name'}, 0, 30;
    # Get Date
    my $dt = substr  $data->{'entry'}->{$arr[$i]}->{'modified'}, 0, 10;
    print "-----------------------------------\n";
    print "| From: $frm, Date: $dt\n";
    print "| $title\n";
}
    print "-----------------------------------\n";
} else {
    print "No new mail...\n";
}
exit 0;
{% endhighlight%}


Output:

{% highlight console %}
lucky@rico:scripts$ ./gmail_atom.pl usename  password
-----> [username] New mails: 20
-----------------------------------
| From: TimesJobs, Date: 2011-11-28
| Resume Criqtiue: Is your resume Effective
-----------------------------------
| From: TimesJobs, Date: 2011-11-14
| Sanket, 7 Mobile, Software Engine.. jobs wait
-----------------------------------
| From: Samrutha, Date: 2011-11-14
| Never Miss a deal. Subscribe Now
-----------------------------------  
{% endhighlight%}

Save the code in '~/scripts/gmail_atom.pl' file. Add following lines in you conky
configurations.

{% highlight text%}
# for account 1
${execi 600 ~/scripts/gmail_atom.pl username1  password1}
# for account 2
${execi 600 ~/scripts/gmail_atom.pl username2 password2}
{% endhighlight%}
		
Final output:

![Conky](/images/gmail-atom-conky.jpg)
