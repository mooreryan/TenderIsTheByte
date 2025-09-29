---
layout: post
title: "How to Clear Ghost Notifications on GitHub"
author: ryan
description: Learn how to remove stuck GitHub ghost notifications using the REST API and curl. Fix phantom alerts and clean up your GitHub inbox fast.
categories: blog
image: "/assets/img/posts/github-ghost-notifications/ghost_notifications.webp"
twitter_share: https://ctt.ac/dlDex
---

Sometimes you may receive a notification in the GitHub UI that you can't get rid of. These "ghost notifications" can arise from being notified by an account or repository that has since been deleted, for example, because it was a [spam account](https://www.bleepingcomputer.com/news/security/github-notifications-abused-to-impersonate-y-combinator-for-crypto-theft/). Regardless of the reason, you won't be able to clear the notification using GitHub's web interface, and you will seemingly be stuck with that little blue dot on your inbox. Luckily, there is a straightforward clear the notification using GitHub's [REST API](https://docs.github.com/en/rest?apiVersion=2022-11-28).

First, you need to [create a token](https://github.com/settings/tokens/new). I created a classic token, and gave it the `notifications` scope. (Be sure to set a reasonable expiration date, or remember to delete the token after you have cleared the ghost notification.)

Once you have created the token, you can use GitHub's notifications API to [mark all notifications as read](https://docs.github.com/en/rest/activity/notifications?apiVersion=2022-11-28#mark-notifications-as-read). You should definitely click that link and read the docs for this endpoint before running the following code to make sure you're familiar with the options! (Alternatively, if you don't want to use curl, you could also try the `gh` command as explained in this GitHub [discussion comment](https://github.com/orgs/community/discussions/174283#discussioncomment-14473564).)

Now, hop into your favorite shell and run something like the following:

{% highlight bash %}
{% raw %}
# Set your generated token to a shell variable.
set GH_NOTIFICATION_TOKEN my_secret_github_token

# Call the GitHub API with curl
curl -L \
  -X PUT \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/notifications \
  -d '{"last_read_at":"2022-06-10T00:00:00Z","read":true}'
{% endraw %}
{% endhighlight %}

In the above code, we set the generated token to a shell variable, and then use curl to hit the GitHub API to mark all notifications as read.

_Note: That code is for [fish](https://fishshell.com/) shell. Adjust the command to set a shell variable as required by your shell of choice. Additionally, check out curl's [manpage](https://curl.se/docs/manpage.html) if you need a refresher on its options._

In my case, I only had the one ghost notification, but if you have some legitimate notifications that you don't want to mark as read, you should check out the [docs](https://docs.github.com/en/rest/activity/notifications?apiVersion=2022-11-28-use) and adjust your command as required.

And that's it! The ghost notifications should be cleared.
