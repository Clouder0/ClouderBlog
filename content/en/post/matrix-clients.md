---
title: "Matrix Desktop Clients"
date: 2022-02-13T15:37:55+08:00
slug: "matrix-clients"
type: post
tags:
categories:
  - Tech
---

Matrix is a decentralized instant message protocol. Besides the official client 「element」, there are several third-party clients available on the market.

## Element

Element is the most popular client as it's the official one. You can use it as a webapp or install its desktop version. It's a cross-platform client with full features.

You can check the details [here](https://matrix.org/docs/projects/client/element).

The UI is elegant and supports light/dark theme. A

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20220213153722-2c5cnta.png)

However, the desktop version is wrapped by Electron, which may be heavy and not that native. I don't want to argue about the pros and cons using Electron apps, but it's always harmless to explore some alternatives.

## Nheko

Nheko is a Qt/C++17 desktop client for Matrix. In terms of the tech stack, we may assume the performance is better, or at least it should come with a native experience.

Like most Qt based applications, Nheko is available on Windows/macOS/Linux. The philosophy behind Nheko is to provide a "mainstream" feeling.

> The motivation behind the project is to provide a native desktop app for Matrix that feels more like a mainstream chat app (Element, Telegram etc) and less like an IRC client.
>

I installed it via AUR and it took me quite a while to compile on my laptop. Flatpak distribution is available, though.

The startpage is kind of simple.

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20220213163118-0iq8pv0.png)

And the support for dark theme is not satisfying.

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20220213163340-jfljaci.png)

SSO login is theoretically available, but nheko failed to open my browser for some reason. I tried several times but in vain, so I gave up. According to what I've experienced, the client is far from refined and is not as decent as element.

And the UI may vary from user to user. There's an official screenshot from nheko's Github readme:

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20220213164727-yprh0nc.png)

Nheko failed to detect my system proxy. I had to use proxychains. I've seen a [Github issue](https://github.com/Nheko-Reborn/nheko/issues/29) about that and it should have been fixed, but I failed anyhow.

## Mirage

Mirage is a fancy, customizable, keyboard-operable Matrix client. Sounds good. However, their Github repo hasn't been updated for almost a year. Seems like the project has been abandoned. Hence, I declined to give it a try.

## Spectral

Spectral is a glossy client for Matrix, written in QtQuick Controls 2 and C++. However, the last commit in their Gitlab repo is 1 year ago. Another abandoned project.

## Fluffychat

> FluffyChat is a Flutter-based client with a simple and clean UI. The focus is on replacing common messenger apps like WhatsApp or Telegram on all major platforms. Find more info on the website: [https://fluffychat.im](https://fluffychat.im/)
>

FluffyChat is a more mature third-party client. It is also a webapp.

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20220213170814-w3jnvzl.png)

I can see the FluffyChat is simpler than element and hide some redundant tech details. I would recommend it to ordinary people who tend to use a traditional IM app.

That is to say, FluffyChat is a good option for your families and friends without tech backgrounds. The developers know what they're doing.

> The focus is on replacing common messenger apps like WhatsApp or Telegram on all major platforms. 
>

## NeoChat

> NeoChat was born as a Spectral fork. It uses [Kirigami](https://develop.kde.org/frameworks/kirigami/) to provide a convergent user interface on desktops (KDE Plasma, GNOME, Windows, and other desktops) and phones (Plasma Mobile and Android).
>

NeoChat is forked from Spectral(that abandoned project) and uses KDE frameworks. I guess that's pretty fine. However, it couldn't adopt my system proxy settings, either. What's worse, proxychains didn't apply to it.

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20220213172000-12bleb3.png)

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20220213171820-2nwz3em.png)

I created an issue in neochat repo. It's a pity that I couldn't go any further now, as without proxy I lose access to matrix.org .

## SchildiChat

SchildiChat is a fork of element.

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20220213175038-gdvs5wj.png)

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20220213175046-0ltl2w2.png)

As far as I'm concerned, the difference between SchildiChat and Element is not that much, though perceptible. Render $\LaTeX$ is a big plus for me.

## Hydrogen

A minimal client. I won't recommend it for daily use as it lacks many features. But if it's sufficient for you, then go for it.

## Conclusion

So many clients are out-dated.  

I think for now, element is still the best option for most people. SchildiChat excels as an element fork if you need the extra features.  
For those who are not familiar with tech stuff, FluffyChat is a good choice.  
If you're using Linux, Nheko and NeoChat are worth trying.
