---
layout: ../layouts/PortfolioLayout.astro
title: "Portfolio"
---

A List of noteable projects, and examples of my work.

## Bepis.uk (This Website)

![bepsuk](https://api.screenshotone.com/take?access_key=yqcPVK7SZYK9Tw&url=https%3A%2F%2Fbepis.uk&format=jpg&block_ads=true&block_cookie_banners=true&block_banners_by_heuristics=false&block_trackers=true&delay=0&timeout=60&response_type=by_format&image_quality=80)

This website serves as my personal homepage, portfolio, and a blog to exercise creative technical writiing.

| [Live](https://bepis.uk) | [Github](https://github.com/bepisdev/bepis.uk) |

---

## Smaller Programming Projects

### Copy

A re-implementation of the GNU `cp` command in Go that adds progress reporting, written to perform large file transfers on my NAS when I learned
running `cp -R` on _10tb_ of data from one HDD to another over an SSH connection meant I was unable to tell if the process had hung or not.

| [Github](https://github.com/bepisdev/copy) |

### Protorec

Written during my time as an electronics technician working on medical assistance alarms and nurse call systems. Protorec acts as mock
reciever endpoint for various reporting protocols these devices use to send alerts like SIA-CID, and MQTT. Used to test these devices during repair and provisioning.

| [Github](https://github.com/bepisdev/protorec) |

### Wikara

A lightweight Wiki engine built as a learning exercise, never intended for production use.

| [Github](https://github.com/bepisdev/wikara) |

### reindent-buffer.el

A plugin for the Emacs text editor. Provides a single command that will safely format the document in the current buffer, simillar to the Format command in VSCode.

| [Github](https://github.com/bepisdev/reindent-buffer) |

### MiniKV

A small Key-Value store written in Rust. Provides a HTTP interface to retireve and store data.

| [Github](https://github.com/bepisdev/minikv) |
