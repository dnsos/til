WIP

```sh
sudo apt install mailutils postfix
```

Use "Local Only" when prompted.

Add to unattended-upgrades config

```
Unattended-Upgrade::Mail "your@email.com";
Unattended-Upgrade::MailOnlyOnError "false";
Unattended-Upgrade::MailReport "on-change";
```
