---
title: "Email Cleaner: Clean tracking links and pixels from email newsletters"
type: post
date: 2021-02-28T12:00:00+08:00
toc: true
tags: ["privacy", "untracker"]
---

*(UPDATE @2021-03-12: There is a [follow-up post](/blog/untracker-remove-tracking-from-email/))*

I'm supposed to be blogging instead of doing ["app startup"](/blog/app-startup-vs-content-startup/) things but I got sidetracked and built another side project (again).

I've been getting into email newsletters. Over the last month, I've gone from zero newsletters to eleven of them. My inbox is now overflowing with newsletters. Maybe I'll have to cut back but that's a story for another time.

I have a new pet hate: Links in email newsletters.

They are user-hostile. Substack is the worst. Their URLs look like this:

http://email.substack.com/c/ejXDKM2OhCAMgJ9muI3hT3QOHPayr2EKVIesgwbquL794njbhEIhNP36eSCclnzYdSnEzm2gY0WbcC8zEmFmW8E8xGBlb1pljGTBdsJpLVksw5gRXxBnS3lDtm5ujh4oLulTIVvJnjbgCKC51hxawbuuxzCCcwHAo0CprrawhYjJo8U35mNJyGb7JFrLTX3d5Hdd-743CWjL2PjlVR8gU_qZLpoGLdre3LkUd87Vw9wrnZX1ymUN3XbtoxENjIYbFKBVq3wrxlF47oLolZNKonrcNC-bKwT-RzT4u1aKRBHmd8S99mTZOkwTQRL153TOfZKcYw_1fG0p0jFgAjdjuIzQJfbjaJgwYa7CwwBkhRGq75Xu-4fpLgVVmZKdFNUzqxxhqVXJ_uP4AyNKkNQ

What is that, Substack?! Is that really a URL with 400+ characters?!

I have an ingrained habit of hovering over links. This allows me to see where it links to and decide whether to click on it.

It's a good habit.

It's also a security "best practice". Hovering over a link helps to check whether it's a phishing link.

Email newsletters have broken this. I have no idea where these user-hostile links go. For their own self interest, mailing list service providers have denied me a basic tenet of the web: Knowing what I'm clicking on before I click.

It's also demeans the newsletter. A newsletter which purports to send me a 'list of interesting links' is just sending me a list of crap because I'm not clicking on those I-have-no-idea-where-it's-going links.

If the link was something like http://www.google.com/url?sa=t&url=https://www.wikipedia.org/&usg=AOvVaw3ay7vaEtH0yTTYdDmrvinX I wouldn't be as peeved because I can see where it links to.

But no, I don't think they'll compromise.

Not happy.

I set about trying to make the situation palatable. I asked [here](https://news.ycombinator.com/item?id=26188976), [here](https://www.reddit.com/r/SomebodyMakeThis/comments/ln4dep/smt_hover_over_links_in_email_newsletters_and_see/), and [here](https://www.indiehackers.com/post/17a17962db) but I didn't find anything satisfactory.

So I built my own solution.


## What is it?

I hooked a script up to an email address. Upon receiving an email, the script will replace tracking links with their destination URL, remove tracking pixels, and send it back to the sender. 

Then, I set up a gmail filter to forward my newsletters to the script.

Yay!

Now, when I receive an email newsletter, gmail forwards it to the script. The script sends it back to me with nice clean URLs. Once again, I can see where links lead to before I decide whether to click.

Tracking parameters (ie. `utm_*`) and tracking pixels are removed as well. (I don't care strongly about this but it was easy to add so I did.)


## Email Cleaner

For lack of a better name, I'm calling it **Email Cleaner**. **It cleans crap from email newsletters**.

(Alternatively, I considered calling it '**UntrackMe: Remove tracking from email newsletters**'. Which sounds better?)

I'm letting others try it. Maybe other people find it useful too. If there's enough interest, I'll turn it into a proper side project.


## Getting started

To try Email Cleaner, forward an email newsletter to `email-cleaner@bengtan.com`.

In a minute or two, you should get the newsletter sent back to you (from `no-reply@bengtan.com`) with cleaned links and tracking pixels removed.

If you decide you like it, you can set up your email program to automatically forward email newsletters to `email-cleaner@bengtan.com`.

How you do this depends on your email program so you'll have to work it out youself.

Note: Please don't set up your email program to automatically delete after forwarding. Since this is a new service, it's a good idea to retain the original emails in case something goes wrong.

### Forwarding loops

If your automatic forwarding configuration mistakenly forwards the cleaned email (from Email Cleaner) back to Email Cleaner, it will detect this and send you an email with the message "Infinite loop detected". If you get such an email, please adjust your forwarding configuration so you don't forward cleaned emails to Email Cleaner.

If you use gmail (like I do), this can be accomplished by adding the condition `-subject:[email-cleaner]` to your forwarding filter.


## How it works

Email Cleaner works as follows:

* It scans the email for links which match the following regular expressions:

```js
	'\.list-manage\.com/track/click',
	'//click\.convertkit-mail\.com/',
	'//email\.substack.*/c/',
	'//apple\.co/',
	'//t\.co/',
```

* If a link matches, then it's a tracking link. Email Cleaner crawls the link to see if it's a 3xx redirect. If it is, then the link is replaced by the destination URL.
    * Any URL query parameters which are `mc_cid`, `mc_eid`, or start with `utm_` are also removed.
* Then, Email Cleaner scans the email for tracking pixels. If any are found, it removes them.
* Finally, the email is sent back to the sender.

Email Cleaner is quite safe and conservative. It will only crawl links which are obviously tracking links.

Email Cleaner currently handles three major mailing list service providers (MailChimp, ConvertKit, Substack). I'd happily add more. Just let me know.

Note that Email Cleaner doesn't magically bypass the tracking links. Instead, it triggers all of them from a data centre somewhere. It's privacy-by-obfuscation instead of privacy-by-abstention. But it does prevent your web browser/IP address/location from being leaked.


## Benefits

* Email newsletters are more user friendly. (I'm more inclined to click on a link if I see it's a reputable website.)
* Helps detect phishing attacks.
* Better user privacy.
* Plain text emails are readable again.


## Plain text emails

The last benefit was an unexpected side effect. 

Whilst writing Email Cleaner, I discovered that plain text versions of email newsletters are completely unreadable.

They look something like this:

> Most agree that the term Artificial Intelligence was codified at a Dartmouth workshop in 1956 [httр://email.substack.com/c/eJxdkc-OgyAQh5-m3Grkr3rgsJd9DQM4WlIFA2O7vv1O2-xlE2CSHzPhy0dwCEsup91zRfY6Rjx3sAmegAiF7BsUE71mvZJcKUa87qgTgpG4mVAAXjYuGssOZN3dEkeLMafPBpOMzFp1wluppHM9Z2FoQQQbnAqd8tx2A7_P2t1HSCNoeEM5cwKy6Blx3R7868F-qkFqjviMK_hom1ym2rrqmr5twVfecTZHLs9tziuJmrWMtqy6kJ0cGtrYoFoF1Aou-ShpCHRsnac9d4wz4MNDtNvuNrTjkzbwu1YNCaNd3hGOZsykaAdpQptonZyur2v3dT1tan7tKeJpIFm3gL954I31Q8hMkKBU3N5Y1FRR3vdc9P2guhtABcZZx2ilTKoOn-tW0v90_AEVdJLl]. At a conference five years earlier [httр://email.substack.com/c/eJxdkEtuhDAMhk8zWSLyBBZZVKp6jSghDkTDJCiYody-YdhV8kO2bPn3N1qEKZdTr3lDcgWD5wo6wbEtdQVEKOyoUMY4WdEbLY0RbLId90oJFus4F4DNxdViOYDth19jcBhzek8ILdjNDk5xrQxozWenoTNmkL2cvQoimFaZz7PumCKkABYeUM6cgK32hrjXi_y6iG9aIW_7QUS3WJGQm1wWSv2aXwVvMd1jWq7O5wOvmwsUQL26NF3_7qiNRStawVtBW-lODw1v3GxaA9wpqWUgxpmH1k-8l15IAXK4qLYevqILd97Az05sCaNbHxGeTcisWA9pQZc4dS4vG5RuLxkj1e1IEc8RkvMrTB9P-NH9NjcukKDQN0yjQ8sNl30vVd8PpvuIIZFSdIKTfUYcU6apZP9x_AIL9Jrk], however, the concepts were discussed in detail, just not named. 

Is that supposed to be readable?! (It's not just Substack. The other providers are almost just as bad.)

Email Cleaner changes it to:

> Most agree that the term Artificial Intelligence was codified at a Dartmouth workshop in 1956 [httрs://en.wikipedia.org/wiki/Dartmouth_workshop]. At a conference five years earlier [httрs://computerhistory.org/blog/thinking-about-machines-and-thinking/], however, the concepts were discussed in detail, just not named. 

which is moderately readable.

It would be even better if it was:

> Most agree that the term Artificial Intelligence was codified at a Dartmouth workshop in 1956 [0]. At a conference five years earlier [1], however, the concepts were discussed in detail, just not named. 
> 
> [0] httрs://en.wikipedia.org/wiki/Dartmouth_workshop<br />
> [1] httрs://computerhistory.org/blog/thinking-about-machines-and-thinking/

but that's a story for another time. (I don't know if it's worth the effort to do this. Do many people still read emails in plain text?)


## Caveats

Please note that Email Cleaner is only a proof of concept. It's an experiment. I wouldn't even call it an MVP.

It's useful to me, but I don't know if it's useful for others. I'd like to find out.

I don't know how long Email Cleaner will last since:

* It's vulnerable to spam (There's no authentication or access control), and
* It's not scalable (It's running on a tiny server).

Once the spam bots arrive, or too many people use it, it'll probably have to stop.

Or if there's sufficient interest, I'll rewrite it into a proper side project.


## Privacy

Please don't forward any personal or private emails to Email Cleaner. Only forward publicly available email newsletters (Paid newsletters are okay).

All received emails go to Email Cleaner's inbox. I hope to accumulate more test data with which to conduct further research on mailing list service providers and improve Email Cleaner.

There's not really a privacy policy. This is an experiment.

If you forward emails to Email Cleaner, I consider that as your opt-in.

Anything you forward to Email Cleaner could be used to improve Email Cleaner.

OTOH, I don't know anything about you anyway. All I have is an email address and what email newsletters you subscribe to (whatever can be inferred from that).



## Feedback

Since this is a new and experimental service, please don't expect it to be perfectly reliable nor correct. If you forward an email to Email Cleaner and don't get a reply in a few minutes, maybe something broke. Please get in touch with me and I'd be happy to look into it.

If you have any questions, suggestions, criticisms, etc. &mdash; please also let me know. I'd be happy to talk.

I hope you like Email Cleaner! Thanks for reading!

