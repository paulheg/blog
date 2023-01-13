---
layout: post
title:  "Easy Telegram notification integration"
---

You want to receive notifications on your phone whenever whatever? ✅

You are able to send http requests? ✅

Then you are at the right place.

1. Go to [my bot](https://t.me/AlaarmAlaaarmBot).
2. Send `/create` to create a new alert.
3. Set a name and description.
4. You now own a custom URL that is bound to this alert.
If a `GET` or `POST` request is sent to this URL you will receive a notification.
5. Profit

Example `curl` command (switch out the url and `m` to your liking)
```bash
curl -X GET "http://localhost/api/v1/alert/{token}/trigger?m=Hello+World"
```
> will send `Hello World` to all subscribers 

## There are some additional features:
### Add the bot to a group
You can add the bot and the alert to a group using the `/invite` command.
Then select `Group` when asked what kind of invite link you want.
If you press on the link, you will get asked to specify the group to add this bot to.
You have to be admin in this group to work.

### Invite single users
You don't have to create a group to add multiple users to your alert.
Just use the `/invite` command and select `Private`.
Share this link with people that you want to add.

### Send Files as a notification
To send a file set the *Form Value* `file` and send a `POST` request to the url.
The file will then be forwarded to Telegram to all subscribed users or groups.

Example `curl` command:
```bash
curl -X POST "http://localhost/api/v1/alert/{token}/trigger?m=I+sent+you+my+code+please+respond" -F "file=@test.jpg"
```