---
title: "Investigating Substack/Mailgun tracking links"
type: post
date: 2021-11-08T12:00:00+08:00
---

In a previous post, I looked at the [tracking links of three major email newsletter providers](/blog/whats-in-email-tracking-links-and-pixels/) and deconstructed two of them. The third, Substack/Mailgun, stubbornly resisted deciphering. Until now.
<!--more-->
After the previous post, a reddit user [helped work it out](https://www.reddit.com/r/programming/comments/nppkeg/whats_in_email_tracking_links_and_pixels/).

A tracking link like this:

```
http://email.substack1.exponentialview.co/c/eJxdkEuOhCAQhk_TLI1VIOqCxWz6GoZ
HaZOx0WDZPc7pB9vdJARSCfU_Pm-ZpiUfZl02Fuc18LGSSfTeZmKmLPaN8hCDwU43UmsUwbTglE
IRt2HMRE8bZ8N5J7Hubo7eclzSZwMbFA8DUtV9b0MgqJUf0bmuRtQBGvQtBXfZ2j1ESp4MvSgfS
yIxmwfzepNfN7yXw-945qn88iyT_S3GIhqsEYpaUW7apq-gsqOuNYFVspG-gXEEX7sAnXQokWR_
U_W2u42t_4aKftbilDja-RXpXbRFNo7SxDZB-Tmd3U7Hs9pQ3ueeIh8DJetmCldrvuB9OAwTJco
FahgsG9Agu06qrut1e9UsWCS2CIWlKDnCUraS-ZfjD2RriM4
```

looks like a bunch of gibberish but turns out to contain data as a base64url encoding of a zlib stream. In other words, data is embedded into the tracking link by (zlib) compressing it and then (base64url) encoding it.

Using [CyberChef](https://gchq.github.io/CyberChef), I decoded the blob into this query-like string (pretty-formatted for readability):

```
category=post&
email_generated_at=1613883488967&
is_freemail=true&
publication_id=2252&
post_audience=everyone&
post_id=32721653&
pub_community_enabled=true&
user_id=28653662&
post_type=newsletter&
subdomain=exponentialview&
l=http://twitter.com/azeem&
r=[redacted]&
d=71b442&
h=134099adde104cf2bb80226d152c7edb&
i=20210221045759.1.af606e1a4353c51ff1c0bd183b232e39@substack1.exponentialview.co&
t=post
```

Let's break this down. Previously, I said:

> I speculate you are sending to Mailgun:
>
> * The destination URL,
> * Whatever is in `X-Mailgun-Variables`,
> * And some other unknown stuff which is taking up the rest of the 90 characters in the blob.

It turns out I was approximately correct.

The destination URL (ie. `http://twitter.com/azeem`) is contained in the field `l`. All of the variables in the `X-Mailgun-Variables` email header are embedded.

The remaining fields:

```
r=[redacted]&
d=71b442&
h=134099adde104cf2bb80226d152c7edb&
i=20210221045759.1.af606e1a4353c51ff1c0bd183b232e39@substack1.exponentialview.co&
t=post
```

contain an email address in `r` (which I've redacted), and some seemingly randomly generated identifiers (Probably some internal Mailgun stuff). These remaining fields account for the 'rest of the 90 characters in the blob'.

Investigation complete. Now we know how to find out exactly what is contained in a Substack (or any other Mailgun retailer) tracking link.


## Discussion

Some random thoughts.

### Why are Substack tracking links so bulky?

Two reasons.

1. Mailgun embeds a bunch of data in there.
2. Substack embeds even more data in there (more than double). Do they really need to embed so much?

### Bypassing tracking

Someone should write software that intercepts these tracking links and redirects to the destination link instead. It would be a step towards better privacy.

This post tells how to do it for Mailgun-based email providers and the previous post does the same for ConvertKit.

(One day, I'll put this feature into email-untracker but it's hardly likely to get widespread usage there.)

### Other newsletter or 'marketing email' providers

Mailchimp, ConvertKit and Substack aren't the only service providers which use tracking links. They're just the ones I've encountered recently.

Some other providers have been suggested to me for investigation as well. I'd like to deconstruct them but I'm a bit busy and not sure when or if I'll have the time. We'll see.

### Base64 with 0% overhead

Previously, I said:

> They managed to do base64 with no overhead??

Yes, they did.

The overhead of base64 is much maligned. In this case, you can get rid of the overhead by compressing beforehand.

Or just don't worry about it because web browsers and servers generally use compression anyway. Base64 introduces overhead due to the redundancy in the encoding. But this very same redundancy is what allows compression to perform well. In the end, it ends up being about the same.

