---
title: Finding Info about GitHub Users
description: ""
date: 2024-06-20T04:01:23.815Z
preview: ""
tags:
    - privacy
    - api
categories:
    - osint
    - recon
keywords: []
---

## Why?

Recently, I ran into an interesting idea and method for finding information about GitHub users using publicly-available resources. It all started when I wanted to see if I could find an email address tied to a particular GitHub account. I knew that this information and much more could theoretically be exposed via commits and general activity on GitHub, but I wanted to find an easier way to do things. 

Plus, I figured as a pentester, this could be a good way to find information related to my targets, like email addresses, technologies used, any custom software or programs being used, etc. Win-win.

## "Abusing" the GitHub API for Receipts

During my research, I stumbled upon an interesting StackOverflow thread, linked here:  
[StackOverflow: How to Leave a Message for a GitHub User](https://stackoverflow.com/questions/12686545/how-to-leave-a-message-for-a-github-com-user)

The basic information about GitHub users can be found by using the `users/<username>` API endpoint that GitHub provides. This provides all the basic information about a user, including their full name, company, location, email addresses, social media links, etc. Note that this information needs to be included by the user on their profile for it to be visible. Additionally, you can find the documentation on related API endpoints and calls [here](https://docs.github.com/en/rest/activity/events?apiVersion=2022-11-28#list-public-events-for-a-user).

The link for this API endpoint is below (replace the x's with a username):  
[https://api.github.com/users/xxxxxxx](https://api.github.com/users/xxxxxxx)

Interestingly, GitHub also provides an API endpoint called `users/<username>/events/public` that can be used to dump all the information about any "public events" related to a username. 

In case you want to bookmark this for later, I included the link here (replace the x's with a username):  
[https://api.github.com/users/xxxxxxx/events/public](https://api.github.com/users/xxxxxxx/events/public)

Trying it out on my own username, I was able to get all sorts of information in JSON format, including: 
- Pull Requests / Pushes / Forks
- Watched / starred repos
- Created / deleted repos
- Email addresses
- Previous usernames
- Avatars, aka profile pictures

As a nifty bonus, you can also script out code snippets to extract specific information from the JSON output. I put an example script on [JSFiddle](https://jsfiddle.net/9gs78o20/) that can extract related email addresses given a GitHub username with the source code below. It's not perfect, as it can occasionally include other users' email addresses, but we aren't going for perfection here, just a "good enough" method for gathering and pivoting information.

```html
<input id=username type="text" placeholder="github username or repo link">
<button onclick="fetch(`https://api.github.com/users/${username.value.replace(/^.*com[/]([^/]*).*$/,'$1')}/events/public`).then(e=> e.json()).then(e => [...new Set([].concat.apply([],e.filter(x => x.type==='PushEvent').map(x => x.payload.commits.map(c => c.author.email)))).values()]).then(x => results.innerText = x)">GO</button>
<div id=results></div>
```

Finally, I've heard about these two really useful tools that can be used for further investigations on GitHub users, as well as organizations/companies/etc. Check them out if you feel the need to dig a little deeper!  
- [octosuite](https://github.com/bellingcat/octosuite)  
- [gitrecon](https://github.com/GONZOsint/gitrecon)

## But I want my privacy!!

Unfortunately, the sad reality is that your information is probably already out there somewhere, and you can't really do anything about it (unless scrubbing ALL your online activity is an option, but I don't cover this). If you still want to take steps to protect your email address on GitHub for a little bit of privacy, I can recommend the following: 

- Set your [email settings](https://github.com/settings/emails) in GitHub to keep your email address private by going to Settings > Emails (under "Access" section) > enable Keep my email addresses private. This will give you a `noreply` email, in the form of `ID+USERNAME@users.noreply.github.com`.

> If you created your account on GitHub.com after July 18, 2017, your noreply email address for GitHub is an ID number and your username in the form of ID+USERNAME@users.noreply.github.com. If you created your account on GitHub.com prior to July 18, 2017, and enabled Keep my email address private prior to that date, your noreply email address from GitHub is USERNAME@users.noreply.github.com.
For more information, refer to the [GitHub docs](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address).
{: .prompt-info }

- In the same settings section > enable Block command line pushes that expose my email. This will prevent any pushes from the command line that can expose your private email from a commit.
- On your local machines (wherever you use git on the command line), set your global email to reflect your `noreply` email. Make sure your username is up-to-date with your current username too.

Example: 
```sh
git config --global user.email "26102428+tunnelcat@users.noreply.github.com"
git config --global user.name "tunnelcat"
```

> This does NOT change any information or hide email addresses retroactively. Previously authored commits associated with a public email will remain public.
{: .prompt-warning }

