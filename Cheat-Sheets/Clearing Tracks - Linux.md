[[Root 𖣂]] | [[eJPT - Cheat Sheets]] | [[eJPT - README]]

<hr>

```shell
history -c && history -w
# This clears the session history and saves the empty history file

clearev
# Clear event logs (stay away unless specified)

history -c
# Clear the bash history

cat /dev/null > ~/.bash_history
# This is a quick way to wipe the saved command history without deleting the file itself
```

> **Note**
> Linux servers may forward logs remotely (e.g., to a SIEM), so local deletion may not be enough. 
> Aggressive log removal often raises red flags. 
> More advanced attackers may edit logs instead of deleting them to avoid suspicion.
