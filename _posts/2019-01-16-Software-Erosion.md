---
layout:     post
title:      How to deal with software erosion?
subtitle:   Understanding software erosion
date:       2019-01-16
author:     Mr.Humorous 🥘
header-img: img/webApp/header.jpg
catalog: true
tags:
    - Heroku
    - WebApp
---

## 1. What is Software Erosion? - Explained by [Adam Wiggins](https://blog.heroku.com/authors/adam-wiggins)

In 2006, I wrote Catapult: a Quicksilver-inspired command-line for the web. I deployed it to a VPS (Slicehost), then gave the URL out to a few friends. At some point I stopped using it, but some of my friends remained heavy users. Two years later, I got an email: the site was down.

Logging into the server with ssh, I discovered many small bits of breakage:
- The app's Mongrel process had crashed and not restarted.
- Disk usage was at 100%, due to growth of logfiles and temporary session data.
- The kernel, ssh, OpenSSL, and Apache needed critical security updates.

The Linux distro had just reached end-of-life, so the security fixes were not available via apt-get. I tried to migrate to a new VPS instance with an updated operating system, but this produced a great deal more breakage: missing Ruby gems, hardcoded filesystem paths in the app which had changed in the new OS, changes in some external tools (like ImageMagick). In short, the app had decayed to a broken state, despite my not having made any changes to the app's code. What happened?

I had just experienced a powerful and subtle force known as [software erosion](https://en.wikipedia.org/wiki/Software_rot).

## 2. Software Erosion is a Heavy Cost
Wikipedia says software erosion is "slow deterioration of software over time that will eventually lead to it becoming faulty [or] unusable" and, importantly, that "the software does not actually decay, but rather suffers from a lack of being updated with respect to the changing environment in which it resides." (Emphasis added.)

If you're a developer, you've probably built hobby apps, or done small consulting projects, that resulted in apps like Catapult. And you've probably experienced the pain of minor upkeep costs over time, or eventual breakage when you stop paying those upkeep costs.

But why does it matter if hobby apps break?

Hobby apps are a microcosm which illustrate the erosion that affects all types of apps. The cost of fighting erosion is highest on production apps — much higher than most developers realize or admit. In startups, where developers tend to handle systems administration, anti-erosion work is a tax on their time that could be spent building features. On more mature projects, dedicated sysadmins spend a huge portion of their time fighting erosion: everything from failed hardware to patching kernels to updating entire OS/distro versions.

Reducing or eliminating the cost of fighting software erosion is of huge value, to both small hobby or prototype apps, and large production apps.

## 3. How Does Erosion-resistance Work?
Erosion-resistance is an outcome of strong separation between the app and the infrastructure on which it runs.

In traditional server-based deployments, the app's sourcecode, config, processes, and logs are deeply entangled with the underlying server setup. The app touches the OS and network infrastructure in a hundred implicit places, from system library versions to hardcoded IP addresses and hostnames. This makes anti-erosion tasks like moving the app to a new cluster of servers a highly manual, time-consuming, and error-prone procedure.

On Heroku, the app and the platform it runs on are strongly separated. Unlike a Linux or BSD distribution, which gets major revisions every six, twelve, or eighteen months, Heroku's infrastructure is improving continuously. We're making things faster, more secure, more robust against failure. We make these changes on nearly a daily basis, and we can do so with the confidence that this will not disturb running apps. Developers on those apps need not know or care about the infrastructure changes happening beneath their feet.

How do we achieve strong separation of app and infrastructure? This leads us to the core principle that underlies erosion-resistance and much of the value of the platform deployment model: __explicit contracts__.

## 4. Explicit Contract Between the App and the Platform
Preventing breakage isn't a matter of never changing anything, but of changing in ways that don't break explicit contracts between the application and the platform. Explicit contracts are how we can achieve almost 100% orthogonality between the app (allowing developers to change their apps with complete freedom) and the platform (allowing Heroku to change the infrastructure with almost complete freedom). As long as both parties adhere to the contract, both have complete autonomy within their respective realms.

Here are some of the contracts between your app running on the Cedar stack and the Heroku platform infrastructure:
- **Dependency management** - You declare the libraries your app depends on, completely and exactly, as part of your codebase. The platform can then install these libraries at build time. In Ruby, this is accomplished with <u>Gem Bundler</u> and `Gemfile`. In Node.js, this is accomplished with <u>NPM</u> and `package.json`.
- **Procfile** - You declare how your app is run with `Procfile`, and run it locally with <u>Foreman</u>. The platform can then determine how to run your app and how to <u>scale out</u> when you request it.
- **Web process binds to `$PORT`** - Your web process binds to the port supplied in the environment and waits for HTTP requests. The platform thus knows where to send HTTP requests bound for your app's hostname.
- **`stdout` for logs** - You app prints log messages to standard output, rather than framework-specific or app-specific log paths which would be difficult or impossible for the platform to guess reliably. The platform can then route those log streams through to a central, aggregated location with <u>Logplex</u>.
- **Resource handles in the environment** - Your app reads config for backing services such as the database, memcached, or the outgoing SMTP server from environment variables (e.g. `DATABASE_URL`), rather than hardcoded constants or config files. This allows the platform to easily connect add-on resources (when you run `heroku addons:add`) without needing to touch your code.

These contracts are not only explicit, but designed in such a way that they shouldn't have to change very often.

Furthermore, these contracts are based on language-specific standards (e.g., Bundler/NPM) or time- proven unix standards (e.g. port binding, environment variables) whenever possible. Well-written apps are likely already using these contracts or some minor variation on them.

An additional concern when designing contracts is avoiding designs that are Heroku-specific in any way, as that would result in vendor lock-in. We invest heavily in ensuring portability for your apps and data, as it's one of our core principles.

Properly designed contracts offer not only strong separation between app and platform, but also easy portability between platforms, or even between a platform and a server-based deployment.
