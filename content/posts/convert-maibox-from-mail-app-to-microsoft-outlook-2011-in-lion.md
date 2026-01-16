---
title: Convert Mailbox from Mail.app to Microsoft Outlook 2011 in Lion
date: 2012-11-09T07:45:14+0000
categories:
- util
author: Bruno Fernandez-Ruiz
aliases:
- /util/convert-maibox-from-mail-app-to-microsoft-outlook-2011-in-lion/
---

In Mac OS 10.7 (Lion) and 10.8 (Mountain Lion), it's not currently possible to export mailboxes from Mail.app to something that Microsoft Outlook 2011 can import directly. Here's a way to fix it.

After a bit of research, and plenty of forum messages seeing the frustration of folks, including [Microsoft's reluctance to fix Outlook 2011 for Mac](http://support.microsoft.com/kb/2598783) (instead they decided to just disable the feature, see in the release note "Import from Apple Mail is disabled in Outlook on Mac OS X 10.7 Lion"), I found the issue was due to the `FSTypeCode` not being set by Mail.app, which is actually really easy to fix.

First, in Mail.app, I export my mailbox. In my case, after I select my Archive mailbox, I get:

	$ ls -1
	Inbox2007.mbox

From Lion onwards, the Apple mbox format is actually a folder that contains a few files inside:

	$ ls -1 Inbox2007.mbox
	Info.plist
	mbox
	table_of_contents

Where `mbox` is the real mbox file we need to import into Outlook, but if we try now (Import -> Contacts or messages from a text file -> Import messages from an MBOX-format text file) we'll see the mbox file in the Finder is greyed out. This is because the `FSTypeCode` is not set:

	$ mdls Inbox2007.mbox/mbox | grep 'FSTypeCode'
	kMDItemFSTypeCode = ""

What we need to do is to change the code to `TEXT`. We can do this with a simple command (I used `find` since I exported several mboxes at once):

	$ find . -name '*.mbox' -exec SetFile -t 'TEXT' {}/mbox \; -print
	./Inbox2007.mbox

If we now test for the `FSTypeCode`, we verify it's correctly set as `TEXT`.

	$ mdls Inbox2007.mbox/mbox | grep 'FSTypeCode'
	kMDItemFSTypeCode              = "TEXT"

We can now finally go back to Outlook and import the mbox file: Import -> Contacts or messages from a text file -> Import messages from an MBOX-format text file.
