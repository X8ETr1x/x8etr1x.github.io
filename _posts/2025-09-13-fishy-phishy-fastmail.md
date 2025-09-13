## Fishy Phishy: Yet Another Billing Phish

*Wow, how awesome that "Fastmail" gave me a six month heads up that my bill was due.*

<img width="836" height="612" alt="Screenshot from 2025-09-13 02-49-46" src="/images/2025-09-13-fishy-phishy-fastmail-1.png" />


So I receive a message from Fastmail support. It looks like my payments aren't working and I might lose service! Panic ensues! I must resolve this immediately and disregard all reason! 

Wait...my renewal isn't until February...and my credit card hasn't changed. What could possibly be wrong? What's wrong is the fact that this message even made it to my inbox *shakes fist at email*. This, my fellow humans, is a phishing attempt to steal my credentials and payment information. 

*"Yeah yeah Trixie, we all know how this works already"* says the seasoned and grizzled blue teamers. Well, this post isn't for them. It's for those who are tired of being scammed but aren't quite sure how to tell. I'm going to walk through how to use common free tools avialable to anyone, investigate as much as possible, and actions that can be taken.

### The Message

So let's break this down. The attacker is appealing to my emotions. The message is stating that I may end up with a service interruption, which clearly I'm not going to like and may even slightly worry about. The goal is to get me to open the attachment - which claims to be some sort of "payment" thingy. Could be a bill, could be a credit card charge failure, could be a cheese pizza. I'm okay with that last one.

#### Message Headers

The first thing I notice are the [email headers](https://datatracker.ietf.org/doc/html/rfc2076), which are what get displayed to me as who it's "from" and who it's to.

<img width="416" height="212" alt="Screenshot from 2025-09-13 03-06-56" src="/images/2025-09-13-fishy-phishy-fastmail-2.png" />

Apparently the "support team" is haloes@fastmail[.]com...whatever that is.

<img width="435" height="200" alt="Screenshot from 2025-09-13 03-13-37" src="/images/2025-09-13-fishy-phishy-fastmail-3.png" />


Points of interest:
+ The green check mark - which is specific to Fastmail - is missing.
+ The "From" address for Fastmail Support is support@fastmail[.]com, not Halo's Master Chief.
+ The "To" address is incorrect. The phishing message had my old primary email address from before I changed it, which Fastmail would never send anything to at this point.

Cool, so just check the From header an I'm all good right? *Nope*. That header can be set to anything and it would be allowed. The attacker technically could have used support@fastmail[.]com, but it may have been caught by spam filters.

#### Message Body

<img width="795" height="351" alt="Screenshot from 2025-09-13 03-19-06" src="/images/2025-09-13-fishy-phishy-fastmail-4.png" />


How thoughtful of them to make things *convenient*. Give me convenience or give me death. Couldn't they just as easily have shown me how to access payment information in the email message itself? Why a [PDF](https://www.rfc-editor.org/rfc/rfc8118.html) file?

### Break it Down

No, I'm not pulling the cardboard out and jamming to Planet Patrol...*actually, that sounds like a good idea.* Instead, I'm going to dig into the guts of the message content and look for specific information.

#### Raw Message

What you typically see in an email message is a formatted [HTML](https://www.ietf.org/rfc/rfc1866.txt) document so that it is easy to read and looks clean. That also means there's a whole bunch of stuff you can't see by default, like [trackers](https://en.wikipedia.org/wiki/Web_tracking) and [hyperlinks](https://en.wikipedia.org/wiki/Hyperlink) that lie about who they are. There's a nice messy way to see all of that, and it's called the **message source** or **raw message**.

<img width="316" height="551" alt="Screenshot from 2025-09-13 03-28-20" src="/images/2025-09-13-fishy-phishy-fastmail-5.png" />


Welcome to a wall of text. Most of it will be useless to you. However, there is a [little tool](https://mha.azurewebsites.net/) to make it easier to read. I'll copy the entire message contents and paste it in the header analyzer for a pretty output. **Do not do this with sensitive information.**

##### Mail Routing

Email messages have to traverse multiple computers systems as they make there way from start to finish. 

<img width="1899" height="500" alt="Screenshot from 2025-09-13 03-34-31" src="/images/2025-09-13-fishy-phishy-fastmail-6.png" />

We can see here that this message never left Fastmail's servers as these are all [internal IP addresses](https://www.ietf.org/rfc/rfc1918.txt), which means the sending account is a Fastmail account.

##### Return Path

The return path address is what a reply message will populate in the "To" header. 

<img width="344" height="80" alt="Screenshot from 2025-09-13 03-35-13" src="/images/2025-09-13-fishy-phishy-fastmail-7.png" />


Here we can see that it is indeed a Fastmail address, which lines up with the mail routing. However, we can't be sure that's the account's primary email address.

##### Email Message Authentication

[SPF](https://datatracker.ietf.org/doc/html/rfc7208) is a way that organizations can claim which email servers are allowed to send with their domain. Unfortunately, since this is all internal to Fastmail, this automatically passes. The same goes for [DKIM](https://datatracker.ietf.org/doc/html/rfc6376) (confirms message integrity) and [DMARC](https://datatracker.ietf.org/doc/html/rfc7489) (confirms the sender domain is legitimate).

<img width="1897" height="67" alt="Screenshot from 2025-09-13 03-38-49" src="/images/2025-09-13-fishy-phishy-fastmail-8.png" />


#### The Attachment

I am going to say with utmost importance: ***DO NOT OPEN ANY ATTACHMENTS FOR ANY REASON...EVER.*** If you open it, then you might infect your computer which is as fun as watching an ant farm devour your house.

I'm going to very carefully save the attachment without opening it and then upload it to [Hybrid Analysis](https://hybrid-analysis.com/submissions/sandbox/urls), which is a website that will test links and files that you provide. Keep in mind, **anything that you upload is public**. After the analysis is complete, I receive a summary report of what they found.

<img width="1568" height="655" alt="Screenshot from 2025-09-13 03-48-53" src="/images/2025-09-13-fishy-phishy-fastmail-9.png" />

<img width="1284" height="807" alt="Screenshot from 2025-09-13 03-49-37" src="/images/2025-09-13-fishy-phishy-fastmail-10.png" />


As we can see, opening the PDF reveals a clicky button to go "update payment" on my Fastmail account. Can I update it to free? That would be cool. 

There's more! 

<img width="962" height="208" alt="Screenshot from 2025-09-13 03-51-13" src="/images/2025-09-13-fishy-phishy-fastmail-11.png" />

<img width="2285" height="153" alt="Screenshot from 2025-09-13 03-51-50" src="/images/2025-09-13-fishy-phishy-fastmail-12.png" />


Ruh-roh Raggy, a [spearphishing](https://en.wikipedia.org/wiki/Phishing) link! While a phising link is a malicious website sent out to a large audience, a spearphishing link is crafted for and sent to a specific group of people or a single person. In this case, the target group is Fastmail users.

#### The Phishing Website

The report also contains the website links from the PDF file. So using Hybrid Analysis once again, we can now submit that link for analysis.

<img width="760" height="178" alt="image" src="/images/2025-09-13-fishy-phishy-fastmail-13.png" />

<img width="1072" height="845" alt="image" src="/images/2025-09-13-fishy-phishy-fastmail-14.png" />

<img width="1092" height="675" alt="Screenshot from 2025-09-13 03-58-19" src="/images/2025-09-13-fishy-phishy-fastmail-15.png" />


The analysis confirms that the site is malicious, but why? Well, a few reasons:

<img width="1556" height="526" alt="image" src="/images/2025-09-13-fishy-phishy-fastmail-16.png" />

<img width="1285" height="807" alt="Screenshot from 2025-09-13 04-01-08" src="/images/2025-09-13-fishy-phishy-fastmail-17.png" />


That sure does look like the Fastmail login page (from last year) doesn't it? If you look at the address bar, it doesn't say fastmail[.]com. It says "I'm a loser attacker who wants to harm innocent people with my fake website."

### So What Do?

Well, the easiest thing is to simply delete it and go on about your day. However, if you'd like to contribute, here's what you can do.

#### Let Your Provider Know

+ Report the message as phishing using the provider's mechanism.
+ Send an email to the provider's support letting them know the email address it came from and that it was reported. They may reach out with more questions or they may say nothing.

#### Report the Domain

Some [domain registrars](https://en.wikipedia.org/wiki/Domain_name_registrar) - which are the companies that provide website addresses - have a reporting feature for abuse. [ICANN](https://en.wikipedia.org/wiki/ICANN) has [a website](https://lookup.icann.org/en/lookup) where you can look up the abuse contact information.

<img width="977" height="191" alt="Screenshot from 2025-09-13 04-16-46" src="/images/2025-09-13-fishy-phishy-fastmail-18.png" />


Here I can see that Tencent is the registrar, so I sent them an email with enough information to hopefully get them to investigate.

<img width="835" height="1203" alt="Screenshot from 2025-09-13 04-22-12" src="/images/2025-09-13-fishy-phishy-fastmail-19.png" />

That's all she wrote. If you followed along to the end, then congratulations. You've just done your first security investigation. Go have a slice of pizza.
